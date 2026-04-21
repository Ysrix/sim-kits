# Scope memo — CAS close + AP (PE replication)

**Prospect (simulated):** [PE-backed accounting rollup, platform + 2 portcos]
**Date:** [ ]
**Civic team:** Mike (SME), Chris (GTM), [engineer TBD]
**Tier:** Platform ($50K, 90 days) — templated across 3 portcos

## The agent

**Name it in one line.** [e.g. "Close + AP coding agent"]

**What it does.** [2–3 sentences covering: invoice ingestion, GL coding, approval routing, posting, close checklist.]

**What it does not do.** [E.g. "Does not initiate payment. Does not change chart of accounts. Does not post above $X without human approval."]

## Success criteria

| Metric | Today (platform co) | Pilot target |
|---|---|---|
| Close duration | | |
| AP invoices coded / FTE-day | | |
| Miscoding rate | | |
| Bill.com + bookkeeping spend | | |
| Time to add a new portco | | 7 days |

## Inputs

- [ ] Chart of accounts per portco (CSV export from QBO/Xero)
- [ ] 3 months of vendor invoices (50+ per portco)
- [ ] Approval rules (by amount, category, vendor)
- [ ] Bank feed (Mercury/Chase) format
- [ ] Close checklist template
- [ ] Prior-period entries as training examples

## System of record

- **Agent writes to:** [QBO / Xero via API. Read-only until shadow mode passes.]
- **Payment rail:** [Mercury or bank ACH. Agent stages, human initiates payments for MVP.]
- **Audit log:** [Tamper-evident log + PE-sponsor-ready audit packet]
- **Credentials scoped to:** [Per-portco creds, rotated monthly]

## Governance envelope

- **Kill switch:** CFO + head of accounting both have the flag
- **Human-in-loop triggers:**
  - Invoice above $10K
  - New vendor (not in prior 6 months)
  - Unusual GL category for that vendor
  - Confidence below 0.85
- **Audit packet for:** PE sponsor's annual review + external audit

## Team assigned

- Domain SME: Mike Marron
- Commercial lead: Chris Hart
- Lead engineer: [ ]
- Governance: Daniel or Martin

## Replication playbook (Platform tier)

Portco onboarding checklist:
1. Export chart of accounts → load into agent config
2. Map approval rules to agent's rule engine
3. Connect bank feed
4. Connect GL API
5. Load 90 days of prior entries for calibration
6. Run 2 weeks shadow mode
7. Cutover

**Target onboarding time per portco: 7 days after platform-co cutover.**

## Open questions

- [ ]
- [ ]
