# Interview guide — Manufacturing RFQ agent (PE replication)

**Domain expert:** Christiano Teixeira
**GTM interviewer:** Chris Hart
**Time:** 45–60 min
**Goal:** Scope an RFQ response agent at a PE-backed industrial platform company, templated for a second portco with a different ERP.
******Chris & Christiano Ltd.

## Framing for Christiano (read verbatim)

"Run this as a prospect call. You're the GM of a mid-market specialty manufacturer — $80M revenue, 120 employees, SAP shop. PE-owned, 18 months into the hold. I'm Chris, a peer CFO from the sponsor. Stay in role. Use Vale, Petrobras, Xavo context if it fits naturally but keep answers grounded in a mid-market shop, not a global logistics network."

### Size of company, and general facts
1. We have a sales account management team of 25 people, which is split by region, type of customer (OEM vs. other).

## Questions

### Pain
1. How many RFQs do you get per week? Per month? - 150 per month
2. Average response time today? - 3 business days
3. What's your win rate? What do you believe it could be? - current 25% / we think it could be > 50%
4. Who touches an RFQ before you quote? Sales, engineering, ops, finance — how many hands? - sales enablement receives the RFQ, checks that we have an existing customer relationship, routes to operations for comparison of RFQ materials to our product catalogue and then inventory checks for in stock items and lead time for out of stock materials. Finance reviews operations proposed RFQ response for compliance with margin goals, shipping costs and general compliance before sending. Total of 3 groups involved.
5. When was the last time you lost a deal on response time alone? - we lose deals every month due to the volume of requests and speed to answer. Often we will prepare and send the RFQ back, but never hear anything from the customer.

### Current state
6. Where do RFQs come from? Email, portal, EDI, mix? - email, website portal (the portal captures limited data - customer name, contact email, phone number, location and then we have the ability for the customer to load a file and one free text field).
  We do use EDI (Electronic Data Interchange) in manufacturing is the computer-to-computer exchange of business documents—such as purchase orders, invoices, and shipping notices—in a standard format for RFQs. It streamlines supply chains by eliminating manual, paper-based processes, reducing errors, and improving speed. 
  Usage Examples in Manufacturing: EDI 840 - request for quote.
  Automated Procurement: Sending EDI 850 Purchase Orders to suppliers to automatically order raw materials.
  Just-in-Time (JIT) Scheduling: Using EDI 830 (Planning Schedule with Release Capability) to communicate forecasts and shipping schedules, reducing inventory holding costs.
  Advanced Shipping Notices (ASN): Sending EDI 856, which provides detailed content, packaging, and carrier information before a shipment arrives.
  Invoicing: Submitting EDI 810 invoices directly into a customer's payment system.
  Inventory Management: Utilizing EDI 846 to exchange real-time inventory levels to avoid stockouts
8. How does the spec come in — PDF, CAD, written? - we receive PDFs with part number requests, sometimes emails with parts and quantity requests, and via
    EDI 840: Request for Quote (RFQ) — Initiated by the buyer to ask for pricing, terms, and conditions.
    EDI 843: Response to RFQ — Sent by the supplier to offer prices, specifications, and delivery schedules.
    EDI 832: Price/Sales Catalog — Used to proactively send vendor product data and prices to a partner.
    EDI 841: Specifications/Technical Information — Used to transmit detailed technical data, which may be part of a complex quote
10. Who pulls inventory? Who prices? - operations has a standard price list that is approved by finance in advance, for volume based pricing or customer based agreements with special pricing, that is loaded into SAP at customer level or inventory management layer.
11. How much is tribal knowledge vs. in the system? - SAP has all the pricing information, Bill of Materials for more complicated orders, customer information, etc.

### Success
10. Target response time? - 24 hours or less
11. What % of RFQs could be quoted without a human? - 75% normal requests / 25% unique requests based on quantity, delivery timing or other factors
12. What's the margin floor the agent must respect? - 
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
