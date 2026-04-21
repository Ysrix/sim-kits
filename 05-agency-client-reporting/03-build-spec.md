# Build spec — Agency client reporting agent

## MVP scope for Day 2

1. Load client config (KPIs, thresholds, template).
2. Load DSP data (mock JSON for MVP, or live read from a sandbox).
3. Load CRM data (mock JSON).
4. Compute KPIs per the definition file.
5. Compare to prior period. Flag anomalies per threshold.
6. Cross-check DSP spend vs. CRM pipeline conversions — flag discrepancies.
7. Write narrative commentary per section (overview, by-channel, anomalies, recommendations).
8. Produce report draft (Markdown for MVP; Google Docs/Slides as stretch).
9. Every metric traces to a pull log entry.

## Out of scope
- Live API pulls for MVP — mock data.
- Report delivery — draft only, no send.
- Budget recommendations with spend changes — commentary only.
- Forecasting.

## Stack

- Python 3.11.
- Claude API (sonnet-4-6 for narrative, haiku-4-5 for KPI summaries).
- Markdown output.
- JSONL pull log + audit log.

## Architecture

```
client_config.yaml ──┐
                     ├──► [load data] ──► dsp_data.json, crm_data.json
dsp_mock.json ───────┤                            │
crm_mock.json ───────┘                            ▼
                                         [compute KPIs]
                                                  │
prior_period/*.json ─────────────────────► [compare + flag anomalies]
                                                  │
                                                  ▼
                                         [write narrative]
                                                  │
                                                  ├──► report_draft.md
                                                  ├──► pull_log.jsonl
                                                  └──► audit_log.jsonl
```

## Client config schema

```yaml
client_id: acme_dtc
client_name: "Acme DTC"
period: "2026-04"
kpis:
  - name: ROAS
    formula: "revenue / spend"
    target: 3.5
    anomaly_threshold_pct: 0.15
  - name: CAC
    formula: "spend / new_customers"
    target: 45
    anomaly_threshold_pct: 0.20
  - name: "Pipeline created ($)"
    source: crm
    field: "pipeline_created_usd"
    anomaly_threshold_pct: 0.25
channels:
  - google_ads
  - meta
  - tiktok
template: "template_standard_v1.md"
anomaly_report_required: true
```

## Report section structure (narrative schema)

```json
{
  "report_id": "acme_dtc_202604",
  "sections": [
    {
      "section": "executive_summary",
      "content": "...",
      "kpis_cited": ["ROAS", "CAC"],
      "sources": ["pull_abc", "pull_def"]
    },
    {
      "section": "by_channel",
      "content": "...",
      "kpis_cited": [],
      "sources": []
    },
    {
      "section": "anomalies",
      "content": "...",
      "flags_raised": [{"kpi": "CAC", "change_pct": 0.32, "reason": "Meta CPMs up 28%"}]
    },
    {
      "section": "recommendations",
      "content": "..."
    }
  ],
  "audit": {
    "data_pulls": [...],
    "model": "claude-sonnet-4-6",
    "generated_at": "2026-04-17T12:00:00Z"
  }
}
```

## Human-in-loop triggers

- Any KPI anomaly >threshold — flagged in report, AM reviews before send
- Data gap (missing day, channel)
- New campaign type never seen before
- CRM and DSP disagree on attributed conversions by >10%

## Success check for demo

3 client configs in `inputs/`:
- Acme DTC — clean month, auto-draft passes
- Beta SaaS — CAC anomaly, must flag and explain
- Gamma Brand — data gap on Meta, must flag gap and deliver partial report

Agent must:
- Produce report drafts for all three
- Correctly flag anomalies with reasoning
- Produce pull log that traces every number
- Handle the data-gap case without silently filling in

## Stretch

- Google Docs output (not just markdown)
- Side-by-side vs. prior period charts
- Recommendation section that cites specific actionable moves with expected impact
