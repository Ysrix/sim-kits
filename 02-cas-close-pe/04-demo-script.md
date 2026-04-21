# Demo script — CAS close + AP agent

**Length:** 8 min + 4 min Q&A
**Presenters:** Engineer lead + Chris (GTM)

## Flow

### 1. Scope recap (60s)
One-line agent description. Success metrics from the memo.

### 2. Invoice in, entry out (90s)
Drop a vendor invoice (PDF) into the inbox folder. Show:
- Extraction result
- Proposed GL coding with reasoning
- Staged journal entry
- Audit log

### 3. The auto-approve (45s)
Show a $400 routine invoice auto-approve based on rules. Highlight: agent didn't make the rule, it applied the rule.

### 4. The escalation (90s)
Show an $8,200 invoice route to controller. Show a $15K invoice route to CFO. Emphasize: agent does NOT bypass approvers.

### 5. The flag (90s)
Show an invoice from a new vendor. Agent flags for human review with reasoning: "First invoice from this vendor in 6 months. No prior coding pattern. Recommend manual review." This is the "don't trust the agent blindly" proof.

### 6. Portco swap (90s)
Run the same agent against portco 2's chart of accounts — different GL, different approval rules, different vendor list. Show it works with config-only changes. This is the Platform-tier + PE replication proof.

### 7. What the PE sponsor gets (45s)
Chris shows the audit packet format. Anchors what the $50K buys a PE-backed accounting rollup.

## Q&A prep

- **"Why not QBO's built-in AP automation?"** QBO handles basic automation but not cross-portco consolidation, not audit-grade logging, not PE-sponsor reporting. And it doesn't replace Bill.com.
- **"What about Bill.com themselves adding AI?"** They have, but you're still locked into their workflow and per-user pricing. The agent runs on your stack.
- **"How do we know it codes correctly?"** Shadow mode for 2+ weeks. Human reviews every entry. Agent doesn't post until accuracy + approval checks pass.
- **"What if QBO API breaks?"** Staged entries wait in queue. No silent failure. CFO sees the backlog.
- **"What's the cost after the pilot?"** Optional retainer. $4K–$12K/month depending on portco count. Handover to in-house team at any time.

## Don't
- Don't demo the extraction on a scanned handwritten invoice. Use clean PDFs for the demo.
- Don't skip the portco swap. It's the single most important slide for PE buyers.
- Don't pretend the agent posts. Staged only. Be explicit.
