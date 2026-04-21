# Interview guide — Manufacturing RFQ agent (PE replication)

**Domain expert:** Christiano Teixeira
**GTM interviewer:** Chris Hart
**Time:** 45–60 min
**Goal:** Scope an RFQ response agent at a PE-backed industrial platform company, templated for a second portco with a different ERP.

## Framing for Christiano (read verbatim)

"Run this as a prospect call. You're the GM of a mid-market specialty manufacturer — $80M revenue, 120 employees, SAP shop. PE-owned, 18 months into the hold. I'm Chris, a peer CFO from the sponsor. Stay in role. Use Vale, Petrobras, Xavo context if it fits naturally but keep answers grounded in a mid-market shop, not a global logistics network."

## Questions

### Pain
1. How many RFQs do you get per week? Per month?
2. Average response time today?
3. What's your win rate? What do you believe it could be?
4. Who touches an RFQ before you quote? Sales, engineering, ops, finance — how many hands?
5. When was the last time you lost a deal on response time alone?

### Current state
6. Where do RFQs come from? Email, portal, EDI, mix?
7. How does the spec come in — PDF, CAD, written?
8. Who pulls inventory? Who prices?
9. How much is tribal knowledge vs. in the system?

### Success
10. Target response time?
11. What % of RFQs could be quoted without a human?
12. What's the margin floor the agent must respect?
13. What's the first failure that would kill the pilot?

### System of record
14. ERP — SAP, NetSuite, Epicor, Plex?
15. Where does the quote get saved — ERP, CRM, CPQ?
16. Who has write access today? Who signs?

### Replication (for portco 2)
17. If you bought another shop tomorrow with a different ERP, what'd break?
18. Catalogs: how different across portcos? Pricing rules?
19. Approval norms — same across portcos or culture-driven?
20. What would you commit to for onboarding time per portco?

### Governance
21. What always needs a human? (Non-standard spec, margin squeeze, new customer)
22. Audit — who asks to see why a quote went out?
23. Kill switch — scenario and owner?

## After the call

Chris fills in `02-scope-memo-template.md` within 30 min.
