# Scope memo — Agency client reporting (DSP + CRM)

**Prospect (simulated):** [Performance agency, 30 clients]
**Date:** [ ]
**Civic team:** Brad (SME), Titus (GTM), [engineer TBD]
**Tier:** Standard ($25K, 60 days)

## The agent

**Name it in one line.** [e.g. "Monthly client report drafter"]

**What it does.** [Pulls DSP spend + metrics, CRM pipeline + wins, applies client's KPI definitions, writes narrative + anomaly flags, outputs report draft.]

**What it does not do.** [Does not send reports. Does not decide budget moves. Does not rewrite strategy.]

## Success criteria

| Metric | Today | Pilot target |
|---|---|---|
| Hours per report | | |
| Reports covered | 30 | 30 |
| QA-pass rate (first draft) | | |
| Anomaly catch rate | | |
| Time to add new client | | 1 day |

## Inputs

- [ ] DSP API creds (Google, Meta, TTD — read-only)
- [ ] CRM export or read-only API (HubSpot/Salesforce)
- [ ] KPI definitions per client (JSON or YAML)
- [ ] Prior 3 months of reports per client (calibration)
- [ ] Report template (branded, per client or shared)
- [ ] Anomaly threshold config per KPI

## System of record

- **Agent writes to:** [Report draft doc — Google Docs/Slides via API, or Notion]
- **Audit log:** [Every data pull, timestamp, source, value]
- **Credentials scoped to:** [Read-only across all DSPs and CRM. No write access anywhere.]

## Governance envelope

- **Kill switch:** Ops lead + AM director
- **Human-in-loop triggers:**
  - KPI change >N% vs. prior period (per client config)
  - Data gap (missing day, missing channel)
  - Conflict between DSP and CRM for same metric
  - New campaign/metric not in prior reports
- **Audit:** Every number in every report traces to a pull log entry. Client question answered in seconds, not hours.

## Team assigned

- Domain SME: Brad
- Lead engineer: [ ]
- GTM sponsor: Titus

## Replication

Template + per-client config. Adding a new client = load KPI def + anomaly thresholds + 3 months calibration data. Target: 1 day.

## Open questions

- [ ]
- [ ]
