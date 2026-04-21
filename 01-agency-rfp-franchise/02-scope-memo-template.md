# Scope memo — Agency RFP + franchise co-op compliance

**Prospect (simulated):** [Franchisor name, e.g. "NationalBrand Franchising, 220 locations"]
**Date:** [ ]
**Civic team:** Brad (SME), Titus (GTM), [engineer TBD]
**Tier:** Platform ($50K, 90 days) — replicated across 220 franchisee submissions

## The agent

**Name it in one line.** [e.g. "Co-op ad compliance reviewer"]

**What it does.** [2–3 sentences. Input → decision → output.]

**What it does not do.** [Explicit boundaries. E.g. "Does not approve spend above $X without human. Does not write to the reimbursement ledger."]

## Success criteria

| Metric | Today | Pilot target |
|---|---|---|
| Avg review time | | |
| % auto-approved (agent confident) | | |
| % flagged for human | | |
| False-approval rate | | |
| Cost per review | | |

## Inputs (from the prospect)

- [ ] Brand guidelines (PDF or link)
- [ ] Co-op program rules
- [ ] Sample submissions — 20 approved, 20 rejected, 10 edge cases
- [ ] Rejection taxonomy (if exists)
- [ ] Approver sign-off list

## System of record

- **Agent writes to:** [e.g. franchisee portal status field + Slack notification]
- **Audit log lands in:** [e.g. S3 bucket + queryable DB]
- **Credentials scoped to:** [which systems, which actions]

## Governance envelope

- **Kill switch:** [who can trigger, how fast]
- **Human-in-loop triggers:** [which conditions]
- **Audit packet for:** [who reviews — legal, brand team, franchisor GC]

## Team assigned

- Domain SME: Brad Webb
- Lead engineer: [ ]
- Governance/security: [Martin or Daniel as needed]
- GTM sponsor: Titus

## Timeline

| Phase | Days | Deliverable |
|---|---|---|
| Scope (this memo) | Day 0 | Signed scope |
| Build on prospect stack | Day 1–45 | Shadow-mode agent |
| Shadow mode | Day 46–75 | Accuracy report vs. human reviewer |
| Cutover | Day 76–90 | Production + audit packet + runbook |

## Replication (Platform tier)

This agent ships as a template. Replication playbook covers: brand-guideline loader, rejection taxonomy config, portal-connector swap, approver routing. Target: 2nd franchisee vertical in 30 days post-cutover.

## Open questions

- [ ]
- [ ]
- [ ]
