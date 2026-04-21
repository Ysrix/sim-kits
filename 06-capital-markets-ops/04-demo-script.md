# Demo script — Capital markets ops (trade break reconciliation)

**Length:** 8 min + 4 min Q&A
**Presenters:** Engineer lead + Chris + Daniel

## Flow

### 1. Scope recap (45s)
5,000 trades/day. X% break rate today. Hours to resolve. Regulatory exposure. Scope: detect breaks, classify, draft resolution narratives — ops staff reviews and resolves.

### 2. Clean match (60s)
Run batch. Show 4 trade pairs matching cleanly. Show:
- Composite key match on trade date, CUSIP, counterparty, direction
- Tolerances applied to price and notional — both within band
- Matched pair logged to audit trail with tolerance values used

### 3. The price break (90s)
Trade pair with price mismatch outside tolerance. Agent classifies as `price_break`. Resolution draft: "Front-office booked at 99.875, back-office settled at 100.125 — 25 bps difference exceeds 12.5 bps tolerance for corporate bonds. Likely cause: stale price on booking, updated by market close. Suggested action: amend front-office booking to match settlement price."
Show:
- Classification reasoning
- Tolerance config cited
- Resolution draft that an ops analyst can act on immediately

### 4. The escalation (90s)
Unmatched front-office trade, $8M notional. No back-office record. Agent classifies as `unmatched_fo`, flags for human review (above $5M threshold). Draft: "No matching settlement record for FO-20260415-007. Notional $8M exceeds auto-classify threshold. Possible causes: late confirm from counterparty, direct deal not yet booked downstream, or SSI routing failure. Recommended: check with JPMC ops desk, verify SWIFT confirm status."
Show:
- Human-in-loop trigger fired (notional threshold)
- Agent does NOT auto-resolve — stages draft for ops review
- Escalation logged to audit trail

### 5. The counterparty mismatch (60s)
Back-office has "JP MORGAN CHASE" vs. front-office "JPMC". Agent identifies as alias mismatch using reference data, classifies as `counterparty_mismatch`, auto-resolves: "Counterparty codes differ — JPMC (front) vs. JP MORGAN CHASE (back). SSI alias table confirms these map to the same legal entity (LEI: 7H6GLXDRUGQFU57RNE97). No action required."
Show:
- Reference data lookup
- Auto-classification below threshold
- Zero ops time spent on a false break

### 6. The audit trail (60s)
Pull the full chain for any break. Show: original trade pair → match attempt → tolerance applied → break classified → resolution drafted → timestamp + model + config version. "Compliance asks for the break file on trade FO-20260415-003. Here it is — 6 seconds."

### 7. What the prospect buys (30s)
Chris: $25K, 60 days. Pilot on one product type (corporates). Expand to govies and structured products via config. Every break traceable, every resolution auditable, ops hours cut by [X]%.

## Q&A prep

- **"What about our existing matching engine?"** The agent sits downstream. It consumes the output of your matching engine (or raw extracts if you don't have one). It adds classification + resolution drafting — the part your ops staff does manually today.
- **"Can it actually resolve breaks, not just draft?"** Not in the pilot. The pilot drafts and stages. Once you trust the classification accuracy, auto-resolution for low-risk break types is a Phase 2 conversation.
- **"What about cross-product netting?"** Out of scope for the pilot. Config-per-product means we can layer in netting logic per product type later.
- **"How does it handle a new counterparty?"** Flags it. New counterparty not in reference data triggers human review. Agent does not guess on counterparty identity.

## Don't
- Don't let the agent auto-resolve the $8M break in the demo. The escalation is the point.
- Don't skip the counterparty alias resolution. "It resolves the false break" is the efficiency proof.
- Don't hand-wave the audit trail. Compliance traceability is the reason regulated buyers sign.
