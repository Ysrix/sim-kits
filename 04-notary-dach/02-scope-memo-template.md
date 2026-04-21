# Scope memo — Notary workflow agent (DACH)

**Prospect (simulated):** [Mid-size notary office, DACH]
**Date:** [ ]
**Civic team:** Martin (SME), Daniel (technical lead), Titus (commercial)
**Tier:** Standard ($25K, 60 days)

## The agent

**Name it in one line.** [e.g. "Matter intake + trust ledger agent"]

**What it does.** [Ingests client submission, validates ID, prepares matter packet, drafts trust ledger entry, produces supervision log.]

**What it does not do.** [Does not sign anything. Does not post the trust entry. Does not make legal judgments. Does not replace the notary.]

## Success criteria

| Metric | Today | Pilot target |
|---|---|---|
| Intake to matter-open time | | |
| Support staff hours per matter | | |
| ID verification completion rate | | |
| Trust entry error rate | | 0 |
| Time to produce supervision log | | seconds |

## Inputs

- [ ] Standard matter templates (real estate purchase, corporate formation, inheritance)
- [ ] ID document types accepted (Personalausweis, Reisepass, eID)
- [ ] Trust account rules and bank details
- [ ] Fee schedule (GNotKG)
- [ ] Supervision log format (per Dienstaufsicht requirements)
- [ ] 10+ completed matters as calibration examples

## System of record

- **Agent writes to:** [Matter management system draft record. Trust entry staged — never posted.]
- **Audit log:** [Tamper-evident, indexed by matter number, retention 10+ years]
- **Credentials scoped to:** [Read-only on ID verification APIs, write staged to matter system, no signing key access]

## Governance envelope (the non-negotiable part)

- **Kill switch:** Any partner. Triggers immediate halt + audit log entry.
- **Always human:**
  - Final notarial act
  - Any legal interpretation
  - Final trust ledger posting
  - Any deviation from template
- **Human-in-loop triggers:**
  - ID verification below confidence threshold
  - Any field missing or ambiguous
  - Matter type not in template library
  - Trust amount above threshold
  - Client flagged by sanctions screen (required — not agent-bypassable)
- **Supervision log:** Every agent action, reasoning, model version, prompt hash, output hash. Signed and timestamped.

## Regulatory framing

- §14 BNotO — notarial supervision duty. Agent is a tool, notary retains supervision.
- BNotK AI guidance (as referenced) — firm must document oversight mechanism. Audit log IS the mechanism.
- GDPR — data minimization, EU-resident processing, ability to purge per-matter.
- DSGVO Art. 22 — no solely-automated decisions with legal effect. Agent flags, notary decides.

## Team assigned

- Domain SME: Martin Riedel
- Technical lead: Daniel Kelleher
- Commercial: Titus Capilnean
- Lead engineer: [ ]

## Timeline

Same 4-phase Standard tier: scope → build → shadow mode (this is where 3+ weeks go) → cutover.

## Open questions

- [ ]
- [ ]
