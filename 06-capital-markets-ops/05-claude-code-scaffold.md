# Claude Code scaffold — Capital markets ops (trade break reconciliation)

## Setup

```bash
cd ~/civic-sim/06-capital-markets-ops
pip install anthropic pyyaml pandas
export ANTHROPIC_API_KEY=sk-ant-...
```

## Directory layout

```
06-capital-markets-ops/
├── agent.py
├── config/
│   ├── corporate_bond.yaml
│   ├── government_bond.yaml
│   ├── counterparty_aliases.json
│   └── kill_switch.flag
├── inputs/
│   ├── front_office_trades.json
│   ├── back_office_trades.json
│   └── expected_outcomes.json
└── outputs/
    ├── matched_pairs.json
    ├── breaks.json
    ├── resolution_drafts.json
    ├── break_report.md
    └── audit_log.jsonl
```

## System prompt

```
You are a trade-break resolution analyst for a fixed-income operations desk. You draft resolution narratives for trade breaks given:

1. The original trade pair (front-office and back-office records)
2. The break classification (price_break, notional_break, unmatched_fo, unmatched_bo, settle_date_mismatch, counterparty_mismatch)
3. The tolerance config that was applied
4. Reference data (counterparty aliases, SSI)

Your job:
- Explain what the break is in plain language
- State the likely root cause based on the data
- Recommend a specific resolution action
- Cite which side likely needs amendment and why

You NEVER:
- Guess at prices or notional amounts. Use only the values provided.
- Assume a counterparty identity without alias table confirmation.
- Recommend auto-resolution for breaks above the escalation threshold.
- Speculate beyond what the data shows.

You ALWAYS:
- Cite the specific tolerance rule that was violated
- State the exact numeric discrepancy
- Reference the trade IDs from both systems
- Flag if the break is reg-reportable (settlement fail at T+1 or later)

Return JSON with fields: trade_ids, break_type, description, likely_cause, recommended_action, escalation_required, reg_reportable.
```

## Core loop

```python
import anthropic, json, yaml, pathlib, pandas as pd
from datetime import datetime

client = anthropic.Anthropic()

SYSTEM_PROMPT = """You are a trade-break resolution analyst for a fixed-income operations desk. You draft resolution narratives for trade breaks given:

1. The original trade pair (front-office and back-office records)
2. The break classification (price_break, notional_break, unmatched_fo, unmatched_bo, settle_date_mismatch, counterparty_mismatch)
3. The tolerance config that was applied
4. Reference data (counterparty aliases, SSI)

Your job:
- Explain what the break is in plain language
- State the likely root cause based on the data
- Recommend a specific resolution action
- Cite which side likely needs amendment and why

You NEVER:
- Guess at prices or notional amounts. Use only the values provided.
- Assume a counterparty identity without alias table confirmation.
- Recommend auto-resolution for breaks above the escalation threshold.
- Speculate beyond what the data shows.

You ALWAYS:
- Cite the specific tolerance rule that was violated
- State the exact numeric discrepancy
- Reference the trade IDs from both systems
- Flag if the break is reg-reportable (settlement fail at T+1 or later)

Return JSON with fields: trade_ids, break_type, description, likely_cause, recommended_action, escalation_required, reg_reportable."""

KILL = pathlib.Path("config/kill_switch.flag")
AUDIT = pathlib.Path("outputs/audit_log.jsonl")

def log_audit(event_type, detail):
    entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "event": event_type,
        "detail": detail
    }
    AUDIT.parent.mkdir(exist_ok=True)
    with AUDIT.open("a") as f:
        f.write(json.dumps(entry) + "\n")
    return entry

def load_trades(path):
    return json.load(open(path))

def load_tolerance(product_type):
    path = pathlib.Path(f"config/{product_type}.yaml")
    if not path.exists():
        return None
    return yaml.safe_load(open(path))

def load_aliases():
    path = pathlib.Path("config/counterparty_aliases.json")
    if path.exists():
        return json.load(open(path))
    return {}

def match_trades(fo_trades, bo_trades, tolerance_configs):
    """Match front-office and back-office trades on composite key."""
    matched = []
    breaks = []
    fo_unmatched = list(fo_trades)
    bo_unmatched = list(bo_trades)

    aliases = load_aliases()

    for fo in list(fo_unmatched):
        tol_cfg = tolerance_configs.get(fo["product_type"])
        match_keys = tol_cfg["match_keys"] if tol_cfg else ["trade_date", "cusip", "counterparty", "direction"]

        for bo in list(bo_unmatched):
            if bo["product_type"] != fo["product_type"]:
                continue

            # Check match keys (with alias resolution for counterparty)
            key_match = True
            for key in match_keys:
                fo_val = fo.get(key, "")
                bo_val = bo.get(key, "")
                if key == "counterparty":
                    fo_resolved = aliases.get(fo_val, fo_val)
                    bo_resolved = aliases.get(bo_val, bo_val)
                    if fo_resolved != bo_resolved:
                        key_match = False
                        break
                elif fo_val != bo_val:
                    key_match = False
                    break

            if not key_match:
                continue

            # Keys match — now check tolerance fields
            pair = {"fo": fo, "bo": bo, "breaks": []}

            if tol_cfg:
                for field, tol in tol_cfg.get("tolerances", {}).items():
                    fo_v = fo.get(field)
                    bo_v = bo.get(field)
                    if fo_v is None or bo_v is None:
                        continue
                    if tol["type"] == "days":
                        from datetime import date
                        fo_date = date.fromisoformat(str(fo_v)[:10])
                        bo_date = date.fromisoformat(str(bo_v)[:10])
                        diff = abs((fo_date - bo_date).days)
                    else:
                        diff = abs(fo_v - bo_v)
                    if tol["type"] == "absolute" and diff > tol["value"]:
                        pair["breaks"].append({
                            "type": f"{field}_break",
                            "fo_value": fo_v,
                            "bo_value": bo_v,
                            "tolerance": tol["value"],
                            "diff": diff
                        })
                    elif tol["type"] == "percentage" and bo_v != 0 and diff / abs(bo_v) > tol["value"]:
                        pair["breaks"].append({
                            "type": f"{field}_break",
                            "fo_value": fo_v,
                            "bo_value": bo_v,
                            "tolerance_pct": tol["value"],
                            "diff_pct": round(diff / abs(bo_v), 6)
                        })
                    elif tol["type"] == "days" and diff > tol["value"]:
                        pair["breaks"].append({
                            "type": f"{field}_mismatch",
                            "fo_value": fo_v,
                            "bo_value": bo_v,
                            "tolerance_days": tol["value"],
                            "diff_days": diff
                        })

            # Check counterparty alias (log if resolved via alias)
            if fo.get("counterparty") != bo.get("counterparty"):
                fo_r = aliases.get(fo["counterparty"], fo["counterparty"])
                bo_r = aliases.get(bo["counterparty"], bo["counterparty"])
                if fo_r == bo_r:
                    pair["alias_resolved"] = True
                    log_audit("alias_resolution", {
                        "fo_code": fo["counterparty"],
                        "bo_code": bo["counterparty"],
                        "resolved_to": fo_r
                    })

            if pair["breaks"]:
                breaks.append(pair)
            else:
                matched.append(pair)

            fo_unmatched.remove(fo)
            bo_unmatched.remove(bo)
            log_audit("match_attempt", {
                "fo_id": fo["trade_id"],
                "bo_id": bo["trade_id"],
                "result": "break" if pair["breaks"] else "matched"
            })
            break

    # Remaining unmatched
    for fo in fo_unmatched:
        breaks.append({"fo": fo, "bo": None, "breaks": [{"type": "unmatched_fo"}]})
        log_audit("match_attempt", {"fo_id": fo["trade_id"], "bo_id": None, "result": "unmatched_fo"})
    for bo in bo_unmatched:
        breaks.append({"fo": None, "bo": bo, "breaks": [{"type": "unmatched_bo"}]})
        log_audit("match_attempt", {"fo_id": None, "bo_id": bo["trade_id"], "result": "unmatched_bo"})

    return matched, breaks

def draft_resolution(break_record, tolerance_config):
    if KILL.exists() and KILL.read_text().strip().lower() in ("1", "true", "halt"):
        return {"status": "halted"}

    prompt = f"""Break record:
{json.dumps(break_record, indent=2)}

Tolerance config applied:
{json.dumps(tolerance_config, indent=2)}

Draft a resolution narrative. Return JSON with: trade_ids, break_type, description, likely_cause, recommended_action, escalation_required, reg_reportable."""

    r = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1500,
        system=SYSTEM_PROMPT,
        messages=[{"role": "user", "content": prompt}]
    )
    return json.loads(r.content[0].text)

def classify_break(break_record):
    """Use haiku for fast break classification."""
    r = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=300,
        messages=[{"role": "user", "content": f"Classify this trade break. Return JSON with break_type and confidence.\n{json.dumps(break_record)}"}]
    )
    return json.loads(r.content[0].text)

def needs_escalation(break_record, tolerance_config):
    threshold = tolerance_config.get("escalation", {}).get("notional_threshold", 5000000)
    fo = break_record.get("fo") or {}
    bo = break_record.get("bo") or {}
    notional = max(fo.get("notional", 0), bo.get("notional", 0))
    return notional >= threshold

def run():
    fo_trades = load_trades("inputs/front_office_trades.json")
    bo_trades = load_trades("inputs/back_office_trades.json")

    # Load all product-type tolerance configs
    tol_configs = {}
    for cfg_file in pathlib.Path("config").glob("*.yaml"):
        cfg = yaml.safe_load(open(cfg_file))
        if "product_type" in cfg:
            tol_configs[cfg["product_type"]] = cfg

    matched, breaks = match_trades(fo_trades, bo_trades, tol_configs)

    resolutions = []
    for brk in breaks:
        product = (brk.get("fo") or brk.get("bo", {})).get("product_type", "unknown")
        tol = tol_configs.get(product, {})

        escalate = needs_escalation(brk, tol)
        classification = classify_break(brk)
        resolution = draft_resolution(brk, tol)
        resolution["escalation_required"] = escalate

        resolutions.append(resolution)
        log_audit("resolution_drafted", {
            "trade_ids": resolution.get("trade_ids"),
            "break_type": resolution.get("break_type"),
            "escalation": escalate
        })

    # Write outputs
    out = pathlib.Path("outputs")
    out.mkdir(exist_ok=True)
    json.dump(matched, open(out / "matched_pairs.json", "w"), indent=2)
    json.dump(breaks, open(out / "breaks.json", "w"), indent=2)
    json.dump(resolutions, open(out / "resolution_drafts.json", "w"), indent=2)

    # Write break report
    report = f"# Trade Break Report — {datetime.utcnow().strftime('%Y-%m-%d')}\n\n"
    report += f"**Trades processed:** {len(fo_trades) + len(bo_trades)}\n"
    report += f"**Matched pairs:** {len(matched)}\n"
    report += f"**Breaks detected:** {len(breaks)}\n"
    report += f"**Escalations:** {sum(1 for r in resolutions if r.get('escalation_required'))}\n\n"
    for r in resolutions:
        report += f"### {r.get('break_type', 'unknown')} — {', '.join(r.get('trade_ids', []))}\n"
        report += f"{r.get('description', '')}\n\n"
        report += f"**Likely cause:** {r.get('likely_cause', '')}\n\n"
        report += f"**Recommended action:** {r.get('recommended_action', '')}\n\n"
        if r.get("escalation_required"):
            report += "**⚠ ESCALATION REQUIRED — notional above threshold**\n\n"
        report += "---\n\n"

    (out / "break_report.md").write_text(report)
    return {"matched": len(matched), "breaks": len(breaks)}

if __name__ == "__main__":
    print(run())
```

## What to build first

1. Load front-office and back-office trade JSON files.
2. Implement composite key matching.
3. Apply tolerance rules from YAML config.
4. Classify breaks by type.
5. Draft resolution narrative for one break.
6. Add counterparty alias resolution.
7. Add escalation threshold logic.
8. Run all 10 trade pairs, verify outcomes.
9. Generate break report.
10. Add audit log.

## Debugging tips

- If matching produces too many false breaks, check that counterparty alias resolution runs BEFORE key comparison.
- If tolerance checks fire incorrectly, verify the tolerance type (absolute vs. percentage) — price uses absolute (bps), notional uses percentage.
- If the model invents trade details in the resolution draft, tighten the system prompt and verify you're passing the full break record.
- Settle date comparison: use date objects, not string comparison. "2026-04-17" and "2026-04-17T00:00:00Z" should match.
