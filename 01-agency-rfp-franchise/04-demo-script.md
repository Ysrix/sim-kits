# Demo script — Agency RFP + franchise co-op compliance

**Length:** 8 min presentation + 4 min Q&A
**Presenter:** Engineer lead + GTM (Titus)

## Flow

### 1. Scope recap (60s)
Read the one-line agent description and success criteria from the scope memo. No preamble.

### 2. Happy path (90s)
Run a clean submission through the agent live. Show:
- Input JSON + image.
- Decision output with rule citations.
- Audit log entry.

### 3. The block (90s)
Run a submission with a blocker violation (e.g. competitor trademark). Show:
- Rejection with specific rule citation.
- Human-in-loop flag.
- Audit log captures the decision + reasoning.

### 4. The edge case (90s)
Run an ambiguous submission. Show the agent flagging for human review rather than guessing. Explain why this is the right behavior for the buyer.

### 5. The kill switch (45s)
Flip the kill-switch flag mid-batch. Show processing halts. Show the audit log records the halt.

### 6. Replication (60s)
Swap in a second franchisor's brand rules. Re-run one submission. Show the agent applies the new rules without code changes. This is the Platform-tier proof.

### 7. What a prospect would get (45s)
Titus reads the one-page scope memo and shows the audit packet format. Anchors what the $50K actually buys.

## Q&A prep

- **"What if the agent is wrong?"** Human-in-loop catches low-confidence cases. Audit log lets you review and retrain.
- **"Why not use [existing brand-compliance SaaS]?"** Existing tools don't integrate with franchisee workflows, aren't audit-ready, and charge per-seat across 200+ franchisees.
- **"How do we trust the vision model?"** Confidence threshold + human fallback + audit log. If the model degrades, you see it before it writes to production.
- **"Who owns the agent after 90 days?"** You do. Handover is part of the pilot.

## Don't
- Don't demo a long prompt on screen.
- Don't show model errors without showing recovery.
- Don't skip the kill switch. It's the single most important slide for governance-minded buyers.
