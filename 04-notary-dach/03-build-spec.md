# Build spec — Notary workflow agent (DACH)

## MVP scope for Day 2

1. Ingest client intake (form data + uploaded ID + supporting docs).
2. Classify matter type against template library.
3. Validate ID (check fields, cross-check, produce confidence score — do NOT do real biometric here).
4. Run sanctions screen (mock API for MVP).
5. Populate matter template with client data.
6. Stage trust ledger entry if applicable.
7. Generate supervision log with full reasoning trace.
8. Produce a signed, tamper-evident audit record per action.

## Out of scope
- Real eID reader integration. Mock.
- Real bank API. Stage only.
- Real BNotK system. Produce XNP-shaped output, don't submit.
- Automated legal interpretation. The agent NEVER interprets legal meaning.
- Generating the notarial act itself.

## Stack

- Python 3.11.
- Claude API (sonnet-4-6 — audit-critical path, use the most capable).
- SQLite for matter + audit store (keep it simple for MVP).
- Ed25519 keypair for audit entry signing.
- Mock ID + sanctions services as local JSON.

## Architecture

```
intake.json + id.pdf + docs/ ──► [classify matter] ──► matter_type
                                         │
template_library/ ──────────────┐        ▼
                                └──► [populate + validate] ──► matter_draft.json
id_validator (mock) ─────────────────────► id_check.json
sanctions_screen (mock) ──────────────────► sanctions_check.json
                                         │
                                         ▼
                                   [trust_ledger_staging] ──► trust_entry_staged.json
                                         │
                                         ▼
                             [supervision log + signed audit] ──► audit_log.jsonl
                                                                  + signed_records/
```

## Supervision log schema (the key artifact)

```json
{
  "record_id": "audit_20260417_001",
  "matter_id": "m_2026_0042",
  "timestamp": "2026-04-17T10:22:14Z",
  "actor": "agent",
  "agent_version": "v0.1.0",
  "model": "claude-sonnet-4-6",
  "model_fingerprint": "sha256:...",
  "prompt_hash": "sha256:...",
  "action": "matter_type_classified",
  "inputs_hash": "sha256:...",
  "output": {
    "matter_type": "real_estate_purchase_de",
    "confidence": 0.93,
    "template_version": "re_purchase_v2.1"
  },
  "human_review_required": false,
  "supervising_notary": null,
  "signed_by": "agent_key_202604",
  "signature": "ed25519:..."
}
```

Every agent action gets a record. Every human review gets a record. Every kill-switch event gets a record. Records chain by hash (previous_record_hash field). That's the tamper-evident part.

## Human-in-loop triggers (HARD)

- Matter type classification confidence below 0.90
- ID confidence below 0.95
- Sanctions screen: ANY hit, regardless of score
- Trust amount above €50,000
- Any template field flagged as "requires notary judgment"
- Client language not in supported list
- Any field the template flags required but missing from intake

## Success check for demo

4 sample matters in `inputs/`:
- 1 clean real estate purchase — goes through end to end, notary only signs at final step
- 1 corporate formation with a missing field — flagged for staff
- 1 inheritance matter with a sanctions hit (mock) — halted, audit log captures halt
- 1 matter where client uploads a non-matching ID — ID validation fails, halted

Agent must:
- Process clean matter, produce complete supervision log
- Flag missing-field matter with specific pointer
- Halt sanctions-hit matter with notice
- Halt ID-mismatch matter with notice
- Produce hash-chained audit trail for all four
- Response to kill switch in any phase

## Stretch

- XNP-shaped output for matter export
- Multi-language intake (DE + EN)
- Template versioning: agent must record which template version it used
