# Demo script — Creator-tech brand safety

**Length:** 8 min + 4 min Q&A
**Presenters:** Engineer lead + Titus

## Flow

### 1. Scope recap (45s)
400 campaigns/quarter. X content pieces reviewed per week. Hours per review. Consistency problem — same content, different reviewers, different answers. Scope: score content against brand rules, produce approve/flag/reject with reasoning.

### 2. Clean approve (90s)
Run submission sub_001 (outdoor content for OutdoorCo). Show:
- Brand ruleset loaded (4 rules)
- Per-rule scoring: no competitors, no alcohol, tone aligned, audience quality good
- Overall score 0.92 → approve
- Reasoning: "On-brand outdoor content, proper disclosure, clean history"
- Audit log entry with every rule checked and result

### 3. The flag — alcohol adjacency (90s)
Run submission sub_004. Creator's post shows craft beer in background of camping scene. Agent flags rule R002 (alcohol_visible) at severity `flag_for_review`. Score drops to 0.68.
Show:
- Image signal detection reasoning
- Score falls into flag band (0.30–0.85)
- Agent does NOT reject — stages for human review with explanation
- "Alcohol visible in background frame. OutdoorCo rules classify this as flag_for_review, not instant_reject. Recommend: brand-side review."

### 4. The instant reject (60s)
Run submission sub_006. Creator post includes competitor product (RivalGear logo visible in image). Agent fires rule R001, instant_reject.
Show:
- Keyword match in text AND image signal
- Severity: instant_reject → score 0.0
- Resolution: "Competitor product RivalGear visible in image frame 2. Rule R001 triggered. Recommended action: reject and notify creator with specific rule citation."
- Even instant rejects get staged for human confirmation — agent does not auto-reject

### 5. The creator history check (60s)
Run submission sub_007. Content is clean, but creator has 1 prior violation within 180-day lookback. Agent flags despite clean content.
Show:
- Content score would be 0.88 (approve range)
- Creator history override triggers flag_for_review
- Reasoning cites the specific prior violation and lookback policy
- "Content passes all rules, but creator has 1 violation in past 180 days. Per policy, flagging for manual review."

### 6. The brand swap (90s)
Run submission sub_008 against a second brand config (HealthCo). Same content type, different rules — HealthCo allows alcohol, rejects fitness supplement claims. Show:
- Config swap, no code changes
- Different rules fire, different score
- This is the replication proof — one agent, N brands

### 7. What the prospect buys (30s)
Titus: $25K, 60 days. Cover your top 20 brands from day one, 1-day onboarding for new brands. Every score explained, every decision auditable, consistency across reviewers goes to 100%.

## Q&A prep

- **"What about video content?"** MVP uses transcript + frame descriptions. Phase 2 adds native video frame analysis. The scoring framework is the same — rules don't change, input signals expand.
- **"Will the model get the vibes wrong?"** The model scores against explicit rules, not vibes. If a brand's rules are specific, the scores are consistent. Ambiguity triggers flag-for-review, not a guess.
- **"What if the brand disagrees with a score?"** The audit log shows exactly which rules fired and why. Update the rule, re-score. The disagreement improves the ruleset.
- **"Can creators see why they were rejected?"** The compliance report is brand-facing. Creator-facing messaging is a separate workflow — but the report gives you exactly what to tell them.
- **"How does this compare to GARM/TAG?"** The agent scores against your rules, which can encode GARM categories. It's not a replacement for the taxonomy — it's the enforcement layer.

## Don't
- Don't skip the alcohol-adjacency flag. "It flags nuance instead of binary reject" is the strongest proof of value.
- Don't auto-reject in the demo — always show the human-confirmation step, even for instant_reject signals.
- Don't demo without the brand swap. Replication across 150 brands is the platform economics story.
