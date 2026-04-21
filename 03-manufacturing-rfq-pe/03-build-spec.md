# Build spec — Manufacturing RFQ agent

## MVP scope for Day 2

1. Ingest incoming RFQ (email body + attachment — PDF spec or text).
2. Parse requested items, quantities, tolerances, lead time.
3. Match to catalog (fuzzy match on part number, description, spec).
4. Check inventory availability and lead time. For MVP, availability = on_hand + on_order. If total (on_hand + on_order) is still insufficient for the requested quantity, flag with estimated lead time.
5. Apply pricing rules (customer tier, volume break, margin floor).
6. Draft a quote line-by-line with reasoning.
7. Route based on approval matrix.
8. Audit log every step.
9. Swap portco 2's catalog/rules in via config only.

## Out of scope
- Real ERP writes. Read-only.
- Real CRM/email sends. Draft to JSON only.
- CAD parsing. Text specs only for MVP.
- EDI. JSON/email only.

## Stack

- Python 3.11.
- Claude API (claude-sonnet-4-6 for all calls in MVP; haiku-4-5 optimization for catalog lookup is a stretch goal).
- Simple vector search on catalog (FAISS or in-memory cosine).
- JSONL audit log.

## Architecture

```
rfq.eml / rfq.json ──► [parse] ──► parsed_rfq.json
                                         │
catalog.csv ──────┐                      ▼
pricing_rules.yaml┤              [match + price + check margin]
inventory.json ───┤                      │
customer_tier.csv ┘                      ├──► quote_draft.json
                                          └──► audit_log.jsonl
```

## Parsed RFQ schema

```json
{
  "rfq_id": "rfq_001",
  "customer": "Acme Industries",
  "customer_tier": "tier_2",
  "received_at": "2026-04-17T08:12:00Z",
  "line_items": [
    {
      "requested_part": "SS-316 flange 4in 150#",
      "quantity": 50,
      "needed_by": "2026-05-15",
      "notes": "RF flange, ASME B16.5"
    }
  ],
  "shipping_to": "Houston, TX",
  "parse_confidence": 0.91
}
```

## Quote draft schema

```json
{
  "quote_id": "q_20260417_001",
  "rfq_id": "rfq_001",
  "portco_id": "portco_001",
  "customer": "Acme Industries",
  "line_items": [
    {
      "requested_spec": "SS-316 flange 4in 150#",
      "matched_sku": "FL-SS316-4-150",
      "match_confidence": 0.94,
      "unit_price": 48.20,
      "quantity": 50,
      "line_total": 2410.00,
      "in_stock": true,
      "lead_time_days": 7,
      "margin_pct": 0.32,
      "pricing_rule_applied": "tier_2_volume_break_50plus"
    }
  ],
  "quote_total": 2410.00,
  "margin_compliance": true,
  "approval_required": false,
  "human_review_required": false,
  "agent_reasoning": "All items matched to catalog with >0.90 confidence. Inventory available. Pricing per tier_2 rules. Margin above floor.",
  "audit_trail_id": "audit_xyz"
}
```

## Human-in-loop triggers

- Any match confidence below 0.85
- Margin below floor on any line
- Non-standard spec (freeform description that doesn't match catalog)
- New customer (not in customer_tier.csv)
- Quote total above threshold for auto-draft

## Success check for demo

10 sample RFQs in `inputs/rfqs/all_rfqs.json` (expected outcomes in `inputs/expected_outcomes.json`):
- 6 "clean" RFQs that should auto-draft (5 portco 1, 1 portco 2)
- 2 RFQs requiring approval routing (1 per portco)
- 2 RFQs that must flag for human (new customer, non-standard spec)

Agent must:
- Draft all 5 clean RFQs correctly
- Route 2 correctly for approval
- Flag the 2 edge cases with clear reasoning
- Run the portco 2 RFQ with config swap only

## Stretch

- Lead-time optimization: suggest split shipments if inventory is partial.
- Competitive pricing: flag if pricing is >10% off historical win rate.
