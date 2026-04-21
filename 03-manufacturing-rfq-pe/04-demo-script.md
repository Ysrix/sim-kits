# Demo script — Manufacturing RFQ agent

**Length:** 8 min + 4 min Q&A
**Presenters:** Engineer lead + Chris

## Flow

### 1. Scope recap (45s)
One-line agent description. Current response time vs. target. Who signs quotes today.

### 2. RFQ in, draft out (90s)
Drop a clean RFQ into the inbox. Show:
- Parsing output
- Catalog match with confidence
- Inventory check
- Priced line items
- Quote draft JSON
- Audit log

### 3. The approval route (60s)
Show a quote above the approval threshold — agent drafts it, marks `approval_required: true`, routes to GM. Agent does not send.

### 4. The flag (90s)
Show a non-standard spec: a custom alloy callout the catalog doesn't have. Agent flags. Reasoning: "Spec 'Hastelloy C-276 machined housing, custom bore' does not match any catalog SKU above 0.60. Requires engineering review before pricing."

### 5. The new customer (60s)
Show an RFQ from "NewCo Industries" — no customer tier on file. Agent flags for sales review before quoting.

### 6. Portco swap (90s)
Run the same agent against portco 2 — different ERP export format, different margin rules, different customer tiers. Show one RFQ process end-to-end with config-only changes.

### 7. What the PE sponsor sees (45s)
Chris walks through the audit packet: every quote drafted, who approved, margin compliance, response time, win/loss once known. This is the replication + PE reporting proof.

## Q&A prep

- **"Why not Salesforce CPQ or DealHub?"** Those price quotes once you've decided what to quote. This agent does the analyst work before — parsing, matching, checking availability.
- **"What if the catalog match is wrong?"** Confidence threshold + margin check + human review on edges. Agent never sends — always a human eye before it goes out.
- **"Can this learn from win/loss?"** Not in MVP. Phase 2 retrains pricing confidence based on outcomes. That's a retainer conversation.
- **"How does replication actually work?"** Config + adapter files. No code changes for a new portco with the same ERP family. SAP → NetSuite or SAP → Epicor needs a new adapter, 3–5 days.

## Don't
- Don't demo on a handwritten RFQ. Use clean text/PDF.
- Don't skip the margin check. It's the single most important slide for a GM.
- Don't claim the agent "knows the catalog." It matches against a loaded catalog. Be precise.
