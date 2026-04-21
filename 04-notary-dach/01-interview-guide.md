# Interview guide — Notary workflow agent (DACH)

**Domain expert:** Martin Riedel
**GTM interviewers:** Daniel Kelleher (technical), Titus Capilnean (commercial)
**Time:** 60 min
**Goal:** Scope an end-to-end notary intake-to-trust-ledger agent. This is the hardest audit/governance build of the seven. Treat it that way.

## Framing for Martin (read verbatim)

"Run this like a real prospect call. You're a partner at a mid-size notary office in Hamburg or Munich — 6 notaries, support staff of 12, typical mix of real estate, M&A, corporate, inheritance. You have Bundesnotarkammer connectivity and follow XNP standards. I'm Daniel on tech, Titus on commercial. Stay in role but use your real Westernacher/ENA/ZeuS-beA context. We want to know what a notary firm actually needs, not what the BNotK needs."

## Questions

### Pain
1. How many matters do you open per week? Per partner?
2. Which workflow burns the most support staff time — intake, ID verification, document prep, trust accounting, or filings?
3. Where do mistakes hit you hardest — missed deadlines, wrong trust entries, incomplete KYC?
4. How much of this is "could be a checklist" vs. true legal judgment?

### Current state
5. What software are you on today? Cross-check: Urkundenverwaltung system, RA-MICRO, Acta Nova, DATEV, something custom?
6. How does a client submit documents today — email, portal, paper?
7. How do you do ID verification — video-ident, in-person, Ausweis reader?
8. Trust account — which bank, how do you post entries?
9. ABA Opinion 512 has a German analogue in §§14 BNotO and the BNotK guidance on AI. How does your firm think about AI supervision today?

### Success
10. If this worked, what's the ROI — staff hours, matter throughput, error rate?
11. What's the first thing that would make a partner kill it?
12. What's non-negotiable for your Dienstaufsicht / regulatory review?

### System of record
13. Where does the matter record live? What's authoritative?
14. Trust ledger — what writes to it today, who approves?
15. E2E-encrypted mailbox (beA equivalent for notaries — is it Nomos/eNK?) — does anything touch it?
16. Archival — what has to go to which Landesarchiv and when?

### Governance (the hard part)
17. What must always be a human decision, no exceptions?
18. What's OK to automate with supervision?
19. What audit trail do Dienstaufsicht and BNotK expect — format, retention?
20. Kill switch — who triggers, who's notified, what happens to in-flight matters?
21. Credential rotation — who holds signing credentials, how often rotated, who revokes?

### DACH specifics
22. XNP / XJustiz conformance — where does the agent need to produce conformant output?
23. Fees: how does GNotKG play in — does the agent compute or validate?
24. Language: German + English matters only, or more?

## After the call

Daniel fills in the technical half of `02-scope-memo-template.md`. Titus fills the commercial half.
