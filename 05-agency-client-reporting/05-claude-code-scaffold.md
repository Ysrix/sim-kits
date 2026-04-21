# Claude Code scaffold — Agency client reporting agent

## Setup

```bash
cd ~/civic-sim/05-agency-client-reporting
pip install anthropic pyyaml
export ANTHROPIC_API_KEY=sk-ant-...
```

## Directory layout

```
05-agency-client-reporting/
├── agent.py
├── config/
│   ├── acme_dtc.yaml
│   ├── beta_saas.yaml
│   ├── gamma_brand.yaml
│   └── kill_switch.flag
├── inputs/
│   ├── dsp/
│   │   ├── acme_dtc_google_ads.json
│   │   ├── acme_dtc_meta.json
│   │   └── ...
│   ├── crm/
│   │   ├── acme_dtc.json
│   │   └── ...
│   └── prior/
│       └── ...
└── outputs/
    ├── reports/
    ├── pull_log.jsonl
    └── audit_log.jsonl
```

## System prompt

```
You are a client reporting drafter for a performance marketing agency. You write monthly reports given:

1. Computed KPIs for the current period (provided)
2. Prior period KPIs for comparison (provided)
3. Anomaly flags already raised (provided — DO NOT invent new ones)
4. Client KPI definitions and targets (provided)
5. Client's prior reports as tone calibration (provided)

Your job:
- Write executive summary grounded in the provided KPIs
- Write by-channel commentary
- Write anomaly section ONLY for flags provided
- Write recommendations

You NEVER:
- Invent numbers. Every number in your output must be present in the provided KPIs or pull log.
- Guess at causation beyond what the data shows. If CAC rose, say CAC rose. Speculate on "why" ONLY if the data supports it.
- Add KPIs that aren't in the client's definition.
- Fabricate anomalies.

You ALWAYS:
- Match the client's tone from prior reports
- Cite the source (channel, metric) for every number
- State the period covered explicitly

Return JSON with sections as specified. Every number must have a source in the pull_log.
```

## Core loop

```python
import anthropic, json, yaml, pathlib
from datetime import datetime

client = anthropic.Anthropic()
KILL = pathlib.Path("config/kill_switch.flag")
PULL_LOG = pathlib.Path("outputs/pull_log.jsonl")
AUDIT = pathlib.Path("outputs/audit_log.jsonl")

def log_pull(client_id, source, query, result_summary):
    entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "client_id": client_id,
        "source": source,
        "query": query,
        "rows": result_summary["rows"],
        "metrics": result_summary["metrics"]
    }
    PULL_LOG.parent.mkdir(exist_ok=True)
    with PULL_LOG.open("a") as f:
        f.write(json.dumps(entry) + "\n")
    return entry

def load_client_data(cfg):
    data = {"dsp": {}, "crm": None}
    for channel in cfg["channels"]:
        path = pathlib.Path(f"inputs/dsp/{cfg['client_id']}_{channel}.json")
        if not path.exists():
            data["dsp"][channel] = {"status": "MISSING", "rows": []}
            log_pull(cfg["client_id"], f"dsp.{channel}", "month_to_date", {"rows": 0, "metrics": {}})
            continue
        data["dsp"][channel] = json.load(open(path))
        log_pull(cfg["client_id"], f"dsp.{channel}", "month_to_date",
                 {"rows": len(data["dsp"][channel].get("days", [])), "metrics": list(data["dsp"][channel].get("totals", {}).keys())})

    crm_path = pathlib.Path(f"inputs/crm/{cfg['client_id']}.json")
    if crm_path.exists():
        data["crm"] = json.load(open(crm_path))
        log_pull(cfg["client_id"], "crm", "month_to_date",
                 {"rows": len(data["crm"].get("deals", [])), "metrics": list(data["crm"].get("totals", {}).keys())})
    return data

def compute_kpis(cfg, data):
    # Aggregate spend and revenue across channels
    spend = sum(data["dsp"][c].get("totals", {}).get("spend", 0) for c in cfg["channels"] if data["dsp"][c].get("status") != "MISSING")
    revenue = sum(data["dsp"][c].get("totals", {}).get("revenue", 0) for c in cfg["channels"] if data["dsp"][c].get("status") != "MISSING")
    new_customers = sum(data["dsp"][c].get("totals", {}).get("new_customers", 0) for c in cfg["channels"] if data["dsp"][c].get("status") != "MISSING")
    pipeline = (data["crm"] or {}).get("totals", {}).get("pipeline_created_usd", 0)

    kpis = {
        "spend": spend,
        "revenue": revenue,
        "ROAS": revenue / spend if spend else 0,
        "CAC": spend / new_customers if new_customers else 0,
        "new_customers": new_customers,
        "pipeline_created_usd": pipeline
    }
    gaps = [c for c in cfg["channels"] if data["dsp"][c].get("status") == "MISSING"]
    return kpis, gaps

def flag_anomalies(cfg, kpis, prior_kpis):
    flags = []
    for kpi_def in cfg["kpis"]:
        name = kpi_def["name"].split()[0] if " " in kpi_def["name"] else kpi_def["name"]
        current = kpis.get(name) or kpis.get(kpi_def["name"])
        prior = prior_kpis.get(name) or prior_kpis.get(kpi_def["name"])
        if current is None or prior is None or prior == 0:
            continue
        change = (current - prior) / prior
        if abs(change) >= kpi_def.get("anomaly_threshold_pct", 0.15):
            flags.append({
                "kpi": kpi_def["name"],
                "current": current,
                "prior": prior,
                "change_pct": round(change, 3)
            })
    return flags

def draft_report(cfg, kpis, prior_kpis, anomalies, gaps):
    if KILL.exists():
        return {"status": "halted"}

    prompt = f"""Client: {cfg['client_name']}
Period: {cfg['period']}

Current KPIs:
{json.dumps(kpis, indent=2)}

Prior period KPIs:
{json.dumps(prior_kpis, indent=2)}

Anomalies flagged (these are the ONLY anomalies to write about):
{json.dumps(anomalies, indent=2)}

Data gaps (do NOT interpolate):
{json.dumps(gaps)}

Write a report with sections: executive_summary, by_channel, anomalies, recommendations. Return JSON."""

    r = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=3500,
        system=SYSTEM_PROMPT,
        messages=[{"role": "user", "content": prompt}]
    )
    return json.loads(r.content[0].text)

def run(client_id):
    cfg = yaml.safe_load(open(f"config/{client_id}.yaml"))
    prior = json.load(open(f"inputs/prior/{client_id}_prior.json"))
    data = load_client_data(cfg)
    kpis, gaps = compute_kpis(cfg, data)
    anomalies = flag_anomalies(cfg, kpis, prior)
    report = draft_report(cfg, kpis, prior, anomalies, gaps)

    out_md = pathlib.Path(f"outputs/reports/{client_id}_{cfg['period']}.md")
    out_md.parent.mkdir(parents=True, exist_ok=True)
    out_md.write_text(f"# {cfg['client_name']} — {cfg['period']}\n\n" +
                      "\n\n".join(f"## {s['section']}\n{s['content']}" for s in report.get("sections", [])))

    with AUDIT.open("a") as f:
        f.write(json.dumps({
            "timestamp": datetime.utcnow().isoformat(),
            "client_id": client_id,
            "kpis": kpis,
            "anomalies": anomalies,
            "gaps": gaps,
            "model": "claude-sonnet-4-6"
        }) + "\n")
    return report

if __name__ == "__main__":
    for cid in ["acme_dtc", "beta_saas", "gamma_brand"]:
        print(run(cid))
```

## What to build first

1. Load one client config + one month of mock DSP data.
2. Compute KPIs.
3. Compare to prior, flag anomalies.
4. Draft narrative.
5. Add pull log.
6. Test data-gap path (delete a channel file, confirm agent flags it).
7. Run all three clients.

## Debugging tips

- If the model invents a number, check that your system prompt is strict. Also verify temperature = 0.
- If anomaly commentary overreaches causation, tighten the prompt: "Only attribute causes that are directly visible in the provided data. Do not speculate."
- Avoid passing full day-by-day DSP data to the model. Pre-aggregate, pass summaries. Keeps numbers consistent.
