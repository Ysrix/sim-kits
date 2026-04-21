# Scope memo — Manufacturing RFQ agent (PE replication)

**Prospect (simulated):** [PE-backed industrial platform + 1 bolt-on portco]
**Date:** [ ]
**Civic team:** Christiano (SME), Chris (GTM), [engineer TBD]
**Tier:** Platform ($50K, 90 days)

## The agent

**Name it in one line.** [e.g. "RFQ response drafter"]

**What it does.** [Reads incoming RFQ, matches specs to catalog, checks inventory and capacity, prices per rules, drafts quote, routes for approval.]

**What it does not do.** [Does not send the quote. Does not commit inventory. Does not override margin floor.]

## Success criteria

| Metric | Today | Pilot target |
|---|---|---|
| Avg response time | | |
| % quoted auto-draft | | |
| Margin compliance rate | | 100% |
| Win rate on quoted RFQs | | |
| New portco onboarding | | 14 days |

## Inputs

- [ ] Product catalog (CSV or ERP export)
- [ ] Pricing rules (discounts, volume breaks, customer tiers)
- [ ] Inventory snapshot (nightly sync is fine)
- [ ] Capacity / lead time rules
- [ ] 30 historical RFQs with the actual quotes sent and outcomes
- [ ] Margin floor by product family
- [ ] Approval matrix (by amount, customer tier, non-standard spec)

## System of record

- **Agent writes to:** [CRM quote draft + CPQ staging. Never sends.]
- **Audit log:** [Tamper-evident, query-able by spec/customer/date]
- **Credentials scoped to:** [Read ERP inventory/catalog, write CRM draft only]

## Governance envelope

- **Kill switch:** GM + head of sales
- **Human-in-loop triggers:**
  - Non-standard spec detected
  - Margin below floor
  - New customer (no prior order history)
  - Amount above threshold
  - Confidence below 0.85
- **Audit:** Full trace — what spec was parsed, what catalog match, what price, what rule applied

## Team assigned

- Domain SME: Christiano Teixeira
- Commercial lead: Chris Hart
- Lead engineer: [ ]
- Governance: Daniel or Martin

## Replication playbook (Platform tier)

Portco onboarding:
1. Export catalog → load adapter
2. Load pricing rules → config
3. Connect ERP API (SAP/NetSuite/Epicor adapter)
4. Connect CRM for draft output
5. Load 30 historical RFQs for calibration
6. Shadow mode 1 week
7. Cutover

**Target: 14 days per bolt-on portco after platform-co cutover.**

## Open questions

- [ ]
- [ ]
