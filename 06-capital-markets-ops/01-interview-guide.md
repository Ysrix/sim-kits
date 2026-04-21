# Interview guide — Capital markets ops (trade break reconciliation)

**Domain expert:** Jonathan Smith
**GTM interviewers:** Chris Hart + Daniel Kelleher
**Time:** 45–60 min
**Goal:** Scope a trade-break reconciliation agent for a fixed-income or derivatives ops desk. Leave with enough for a one-page scope memo.

## Framing for Jonathan (read verbatim)

"Run this like a prospect call. You're the Head of Operations Technology at a mid-tier fixed-income shop — 200 people in ops, clearing 4,000–6,000 trades a day across govies, corporates, and structured products. I'm Chris, a peer executive who ran brokerage infrastructure. Daniel's here as technical co-pilot — he built the structured-deal management system at RBS. Stay in role. Use your real operator experience as source material."

## Questions

### Pain
1. How many trade breaks per day, as a percentage of total volume?
2. What does a break cost you — in settlement fails, in ops hours, in regulatory exposure?
3. When a break isn't caught by COB, what happens next? Who gets the call?
4. Last time a break turned into a regulatory event, what went wrong?

### Current state
5. Front-office booking system — Murex, Calypso, Bloomberg TOMS, internal?
6. Back-office settlement — who? Broadridge, internal, outsourced to a custodian?
7. How do you match today — automated matching engine, spreadsheet, eyes on screen?
8. What fields does the match run on — trade date, settle date, CUSIP/ISIN, notional, counterparty, price?
9. Tolerance rules — what's the threshold before a mismatch becomes a break? Is it the same across products?

### The break lifecycle
10. When a break fires, where does it land — a queue, an email, a dashboard?
11. Who triages? What's the first decision — which side is wrong?
12. Average time from break detection to resolution today?
13. What percentage of breaks are the same root cause recurring (e.g. booking errors, SSI mismatches, late confirms)?

### Success
14. Break resolution time — what's the target?
15. What would "auto-resolvable" mean to you? Which break types could an agent close without a human?
16. What does the regulator need to see in your break report? CSDR, SEC 15c6, internal SLA?
17. If a break is still open at T+1 close, what's the escalation path?

### System of record
18. Where does the resolved break get recorded — the matching engine, a case management system, both?
19. Who has write access to mark a break as resolved?
20. Audit trail — what does compliance ask for when they pull a break file?

### Governance
21. Which break types always require a human decision — anything above a notional threshold, counterparty disputes, reg-reportable?
22. Who signs off on the agent's auto-resolution queue before it posts?
23. Kill switch — who has it, and what's the blast radius if it triggers mid-batch?

## After the call

Chris fills in `02-scope-memo-template.md` within 30 min. Daniel validates the technical feasibility of matching logic and tolerance rules.
