# Build spec — Capital markets ops (trade break reconciliation)

## MVP scope for Day 2

1. Load trade extracts from front-office and back-office (mock JSON/CSV).
2. Match trades on composite key: trade date, settle date, CUSIP/ISIN, counterparty, notional, price.
3. Apply tolerance rules per product type (configurable YAML).
4. Classify breaks: exact mismatch, tolerance breach, unmatched (one-sided), late confirm.
5. For each break, draft resolution narrative — what's wrong, likely root cause, suggested action.
6. Produce break summary report (Markdown for MVP; reg-formatted stretch).
7. Every match attempt and break classification logged to audit trail.
8. Kill switch halts processing mid-batch.

## Out of scope

- Live connectivity to booking or settlement systems — mock data only.
- Auto-resolution (agent suggests, human resolves).
- Counterparty communication.
- CSDR penalty calculation — flag only.
- Historical pattern analysis across months.

## Stack

- Python 3.11.
- Claude API (sonnet-4-6 for resolution narratives, haiku-4-5 for break classification).
- pandas for matching and tolerance logic.
- YAML configs per product type.
- Markdown + JSONL output.

## Architecture

```
front_office_trades.json ──┐
                           ├──► [match engine] ──► matched_pairs + unmatched
back_office_trades.json ───┘          │
                                      ▼
tolerance_config.yaml ────────► [apply tolerances]
                                      │
                                      ▼
                              [classify breaks]
                                      │
                                      ├──► breaks.json
                                      ▼
                              [draft resolutions]
                                      │
                                      ├──► break_report.md
                                      ├──► resolution_drafts.json
                                      └──► audit_log.jsonl
```

## Trade record schema

```json
{
  "trade_id": "FO-20260415-001",
  "source": "front_office",
  "trade_date": "2026-04-15",
  "settle_date": "2026-04-17",
  "product_type": "corporate_bond",
  "cusip": "037833AK6",
  "isin": "US037833AK68",
  "counterparty": "JPMC",
  "direction": "buy",
  "notional": 5000000.00,
  "price": 99.875,
  "currency": "USD",
  "trader": "desk_a"
}
```

## Tolerance config schema

```yaml
product_type: corporate_bond
tolerances:
  price:
    type: absolute
    value: 0.125        # 12.5 bps
  notional:
    type: percentage
    value: 0.001         # 0.1%
  settle_date:
    type: days
    value: 0             # exact match required
match_keys:
  - trade_date
  - cusip
  - counterparty
  - direction
escalation:
  notional_threshold: 5000000
  auto_classify_below: 1000000
```

## Break classification taxonomy

| Type | Description | Agent action |
|---|---|---|
| `price_break` | Price outside tolerance | Draft: which side likely correct based on market data reference |
| `notional_break` | Notional outside tolerance | Draft: flag booking discrepancy, suggest amendment side |
| `unmatched_fo` | Front-office trade with no back-office match | Draft: likely late confirm, check settlement queue |
| `unmatched_bo` | Back-office record with no front-office booking | Draft: possible direct deal or missed booking |
| `settle_date_mismatch` | Settlement dates differ | Draft: cite contractual settle convention, suggest correction |
| `counterparty_mismatch` | Counterparty codes don't align | Draft: check alias table, likely SSI mapping issue |

## Human-in-loop triggers

- Notional >$5M on any break
- Unmatched trades still open at T+1 16:00
- Counterparty dispute (both sources claim correct, values differ)
- Product type not in tolerance config
- More than 3 breaks from the same counterparty in one batch (pattern flag)

## Success check for demo

10 trade pairs in `inputs/`:
- 5 clean matches (no break, including 1 near-miss price diff within tolerance)
- 1 price break (outside tolerance)
- 1 notional break above escalation threshold
- 1 unmatched front-office (late confirm scenario)
- 1 settle date mismatch
- 1 counterparty code mismatch

Agent must:
- Correctly match the 5 clean pairs (including 1 price near-miss within tolerance)
- Flag the 1 price break (outside tolerance)
- Escalate the >$5M notional break with narrative
- Identify and explain the unmatched, settle date, and counterparty breaks
- Produce break report + audit log

## Stretch

- Regulatory break report format (CSDR fields)
- Break trend dashboard (this week vs. prior week)
- SSI lookup enrichment (counterparty alias resolution)
- Auto-suggest which side to amend based on market reference data
