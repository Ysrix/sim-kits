# Interview guide — CAS close + AP (PE replication)

**Domain expert:** Mike Marron
**GTM interviewer:** Chris Hart
**Time:** 45–60 min
**Goal:** Scope an agent that runs month-end close + AP at a PE-backed accounting firm's platform company, then templates across 2 portcos.

## Framing for Mike (read verbatim)

"I'm going to run this like a prospect call. You're the CFO of a PE-backed accounting rollup — think Aprio, Citrin Cooperman, Ascend. I'm Chris, a peer CFO from a similar portfolio. Stay in role. Use your real operator experience (Bill.com replacement, Mercury + QuickBooks) as the source material."

## Questions

### Pain
1. How many days is your close today? How many should it be?
2. AP volume per month per portco?
3. What do you pay Bill.com + outsourced bookkeeping per portco?
4. When the PE sponsor asks for portfolio consolidation, how long does it take?

### Current state
5. Which GL does each portco use? (Likely QBO variants, some on Xero.)
6. Who codes AP today? In-house AP clerk, outsourced, mix?
7. Approval chain per invoice type?
8. Bank — Mercury, Chase, Bank of America, mix?

### The Bill.com teardown
9. What specifically does Bill.com do that's non-trivial to replace?
10. What does it do that you don't actually use?
11. What would you need the agent to handle that Bill.com handles today?

### Success
12. Days of close, end state?
13. AP cost per invoice, end state?
14. False-coding rate you can tolerate?
15. What does the PE sponsor need to see in a consolidated report?

### Replication
16. What's same across portcos? Chart of accounts structure, approval norms, vendor types?
17. What's different? GL, bank, approver list, classes/locations?
18. If you template this, what's the per-portco onboarding time you'd commit to?

### Governance
19. Who signs off on agent-coded entries before they post?
20. Audit trail for the PE sponsor's annual audit — what format?
21. Kill switch — who has it, who sees it got flipped?

## After the call

Chris fills in `02-scope-memo-template.md` within 30 min.
