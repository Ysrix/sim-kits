# Build spec — CAS close + AP agent

## MVP scope for Day 2

1. Ingest a folder of invoices (structured JSON for hackathon; PDF extraction as stretch).
2. Extract vendor, date, amount, line items, PO number (if present).
3. Match to chart of accounts, propose GL coding.
4. Route to approval based on rules.
5. Stage a journal entry (do NOT post in MVP — write to staging JSON).
6. Log every action to audit trail.
7. Support a second portco with a different chart of accounts, no code changes.

## Out of scope
- Real GL posting. Stage only.
- Real payment initiation. Stage only.
- Bank reconciliation.
- Multi-entity consolidation reporting.

## Stack

- Python 3.11 or Node 20.
- Claude API (sonnet-4-6 for coding logic, haiku-4-5 for extraction).
- PDF parsing: pypdf or pdf.js (stretch — use structured JSONL for hackathon).
- Config-driven per-portco chart of accounts (YAML/JSON).
- JSONL audit log.

## Architecture

```
invoices.jsonl ──► [extract]  ──► invoice.json
                                       │
portco_config.yaml ─────────────┐      ▼
                                └──► [GL coder + rule engine] ──► staged_entry.json
                                              │
                                              ├──► audit_log.jsonl
                                              └──► approval_queue.json
```

## Config schema (per portco)

```yaml
portco_id: portco_001
name: "Acme Manufacturing"
gl: "QBO"
chart_of_accounts: "inputs/portco_001_coa.csv"
approval_rules:
  - name: "auto_approve_under_1k"
    condition: "amount < 1000 AND vendor in known_vendors"
    action: "auto_approve"
  - name: "route_to_controller"
    condition: "amount >= 1000 AND amount < 10000"
    action: "approval.controller"
  - name: "route_to_cfo"
    condition: "amount >= 10000"
    action: "approval.cfo"
known_vendors: "inputs/portco_001_vendors.csv"
```

## Invoice extraction schema

```json
{
  "invoice_id": "inv_001",
  "vendor": "Acme Supply Co",
  "vendor_id_confidence": 0.95,
  "invoice_date": "2026-04-03",
  "invoice_number": "12345",
  "amount_total": 2400.00,
  "line_items": [
    {"description": "Q2 office supplies", "amount": 2400.00, "gl_proposed": "6200-Office Supplies", "confidence": 0.91}
  ],
  "po_number": null,
  "extraction_confidence": 0.93
}
```

## Staged entry schema

```json
{
  "entry_id": "je_20260403_001",
  "portco_id": "portco_001",
  "invoice_id": "inv_001",
  "debit": [{"account": "6200-Office Supplies", "amount": 2400.00}],
  "credit": [{"account": "2000-Accounts Payable", "amount": 2400.00}],
  "status": "awaiting_approval",
  "approver": "controller@portco_001.com",
  "agent_reasoning": "Vendor 'Acme Supply Co' has coded to 6200 in 12 of last 12 occurrences. Amount within normal range. Auto-routed per rule 'route_to_controller'.",
  "human_review_required": false,
  "audit_trail_id": "audit_xyz"
}
```

## Success check for demo

Given 15 invoices across 2 portcos in `inputs/`:
- Extract cleanly on 14 of 15.
- Correctly code the 12 "normal" invoices.
- Flag the 3 unusual invoices for human review (new vendor, unusual category, above threshold).
- Produce complete audit log.
- Demonstrate swap to portco 2 with different COA — no code changes.

## Stretch

- Close checklist agent: reads checklist, checks which items are done based on GL state, reports remaining.
- Vendor deduplication across portcos.
