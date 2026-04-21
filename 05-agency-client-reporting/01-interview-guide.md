# Interview guide — Agency client reporting (DSP + CRM)

**Domain expert:** Brad Webb
**GTM interviewer:** Titus Capilnean
**Time:** 45 min
**Goal:** Scope a monthly client-reporting agent that pulls from DSPs and CRM, writes commentary, flags anomalies.

## Framing for Brad (read verbatim)

"Run this as a prospect call. You're the COO of a mid-size performance agency — 45 people, 30 clients, mostly DTC brands and B2B SaaS. You spend 2 days per client per month on reporting. I'm Titus, a peer operator. Stay in role."

## Questions

### Pain
1. How many monthly reports go out? How long does each one take?
2. Who writes them — account manager, analyst, director?
3. What's the quality range — best report vs. worst?
4. When a client churns, how often does bad/late reporting factor in?

### Current state
5. Which DSPs — Google Ads, Meta, TTD, DV360, Amazon, mix?
6. CRM — HubSpot, Salesforce, something smaller?
7. Does the agency use Supermetrics, Funnel, Whatagraph, or raw API pulls?
8. What's the output format — Google Slides, Looker, PDF, Notion?

### Success
9. Target time per report, end state?
10. Quality floor — what must every report have, no exceptions?
11. What's the first quality failure that would make a client complain?

### System of record
12. Where does the report get delivered? Email, Slack, portal?
13. Who QAs it today — AM, director, both?
14. What data source is the source of truth if DSP and CRM disagree?

### Governance
15. Client-specific sensitive data — revenue, COGS, pipeline — what does the agent touch, what is off-limits?
16. When a KPI moves unexpectedly, what's the agent supposed to do — write commentary, flag, or both?
17. Audit — does the client ever ask "where did this number come from?" What's the answer today?

### Replication
18. Across 30 clients, how similar are the reports? Template with variables, or fully bespoke?
19. What % of clients could share one template vs. need their own?

## After the call

Titus fills in `02-scope-memo-template.md` within 30 min.
