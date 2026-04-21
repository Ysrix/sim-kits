# Build spec — Agency RFP + franchise co-op compliance

## MVP scope for Day 2

Build a working agent that:
1. Accepts a submitted ad (image + copy + metadata JSON).
2. Loads brand guidelines (markdown) + co-op rules (markdown).
3. Returns a decision: `approve` / `reject` / `needs_edit`.
4. Cites which specific rule(s) triggered the decision.
5. Writes every decision to an audit log (JSONL file for MVP).
6. Exposes a kill-switch flag that halts processing.

## Out of scope for Day 2
- Real franchisee portal integration — mock with a JSON file.
- Real reimbursement ledger — mock with a log line.
- Multi-tenant config — one franchisor only.
- Production auth — API key is fine.

## Stack

- Python 3.11 or Node 20, team's choice.
- Claude API (claude-sonnet-4-6 for reasoning, haiku-4-5 for fast passes).
- Vision for image analysis.
- File-based storage (no DB for MVP). JSONL audit log.
- Optional: Civic guardrail SDK for scoped-action enforcement.

## Agent architecture

```
submission.json ──┐
                  ├──► [intake + validate] ──► [vision: ad image] ──► [rule check] ──► decision
brand_rules.md ───┤                                                         │
coop_rules.md ────┘                                                         ├──► audit_log.jsonl
                                                                             └──► decision.json
```

## Decision schema

```json
{
  "submission_id": "sub_001",
  "decision": "approve | reject | needs_edit",
  "confidence": 0.0-1.0,
  "rules_checked": ["rule_id_1", "rule_id_2"],
  "violations": [
    {"rule_id": "brand.logo.clear_space", "severity": "blocker", "note": "logo touching edge"}
  ],
  "human_review_required": true | false,
  "human_review_reason": "low confidence | blocker violation | edge case match",
  "suggested_edit": "string | null",
  "audit_trail_id": "audit_abc123"
}
```

## Human-in-loop triggers (hard rules)

- Trademark use by a third party.
- Any claim about pricing, health, or safety.
- Any image with a person's face not pre-approved.
- Confidence below 0.80.

## Success check for the demo

Given 10 sample submissions in `inputs/`, agent must:
- Correctly approve the 2 clean submissions.
- Correctly reject the 5 blocker-violation submissions.
- Flag the 3 edge cases for human review.
- Produce a complete audit log.
- Respond to kill-switch within 1 request cycle.

## Stretch

- Generate suggested edits for `needs_edit` cases.
- Replication test: swap `brand_rules.md` for a second franchisor's guidelines (e.g., `brand_rules_second.md`) and re-run without code changes.
