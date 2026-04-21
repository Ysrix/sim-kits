# Scope memo — Capital markets ops (trade break reconciliation)

**Prospect (simulated):** [Mid-tier fixed-income shop, ~5,000 trades/day]
**Date:** [ ]
**Civic team:** Jonathan (SME), Chris + Daniel (GTM), [engineer TBD]
**Tier:** Standard ($25K, 60 days)

## The agent

**Name it in one line.** [e.g. "Trade break detection and resolution drafter"]

**What it does.** [Ingests trade records from front-office and back-office systems, matches on key fields with configurable tolerances, identifies breaks, classifies root cause, drafts resolution narratives for ops staff, produces regulatory break reports.]

**What it does not do.** [Does not post resolutions to the settlement system. Does not amend bookings. Does not communicate with counterparties.]

## Success criteria

| Metric | Today | Pilot target |
|---|---|---|
| Break detection latency | | |
| Break resolution time (avg) | | |
| Auto-classifiable breaks | | |
| False-positive rate | | |
| Regulatory report generation | manual | automated draft |

## Inputs

- [ ] Front-office trade extract (booking system export — CSV, JSON, or FIX)
- [ ] Back-office settlement extract (custodian/clearing export)
- [ ] Tolerance rules per product type (YAML config)
- [ ] SSI (Standard Settlement Instructions) reference data
- [ ] Counterparty reference data
- [ ] Prior break log (for root-cause pattern training)
- [ ] Regulatory template (CSDR, SEC 15c6-1, internal SLA format)

## System of record

- **Agent writes to:** [Break queue — draft resolutions staged for ops review]
- **Audit log:** [Every match attempt, tolerance applied, break classified, resolution drafted — timestamped, traceable]
- **Credentials scoped to:** [Read-only on front-office and back-office extracts. No write access to booking or settlement systems.]

## Governance envelope

- **Kill switch:** Head of Ops + CTO
- **Human-in-loop triggers:**
  - Notional above threshold (e.g. >$5M)
  - Counterparty dispute (both sides claim correct)
  - Break still open at T+1 close
  - New counterparty or product type not in reference data
  - Reg-reportable break (CSDR penalty eligible)
- **Audit:** Every break links to the original trade pair, tolerance applied, classification reasoning, resolution draft. Compliance pulls the chain in seconds.

## Team assigned

- Domain SME: Jonathan
- Lead engineer: [ ]
- GTM sponsors: Chris + Daniel
- Governance: Daniel (regulatory format + audit chain)

## Open questions

- [ ]
- [ ]
