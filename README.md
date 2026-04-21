# Civic 2-day simulation kits

7 exercises. Each kit has 5 files + sample inputs.

## Structure per kit

- `01-interview-guide.md` — GTM touchpoint runs this as a discovery call with the domain expert
- `02-scope-memo-template.md` — fill this in post-interview; one-page scope memo (matches the real product deliverable)
- `03-build-spec.md` — what engineering builds in Day 2
- `04-demo-script.md` — end-of-Day-2 demo structure
- `05-claude-code-scaffold.md` — starter prompts, tool list, agent skeleton
- `inputs/` — sample/synthetic inputs for the MVP build

## The 7 exercises

| # | Exercise | Domain expert | GTM touchpoint | Tier |
|---|---|---|---|---|
| 1 | Agency RFP + franchise co-op compliance | Brad | Titus | Platform |
| 2 | CAS close + AP (PE replication) | Mike | Chris | Platform |
| 3 | Manufacturing RFQ (PE replication) | Christiano | Chris | Platform |
| 4 | Notary workflow (DACH) | Martin | Daniel + Titus | Standard |
| 5 | Agency client reporting (DSP+CRM) | Brad | Titus | Standard |
| 6 | Capital markets ops (FS warm-intro) | Jonathan | Chris + Daniel | Standard |
| 7 | Creator-tech brand safety | Brad | Titus | Standard/Platform |

## Day 1 AM
7 parallel 60-min interviews. GTM runs the call as they would with a prospect. Output: scope memo per exercise.

## Day 1 PM
Scope review across all 7. Cut to what's buildable in one day. Assign engineers.

## Day 2
MVP build. Demo round end of day.

## Ground rules

- Treat the interview as a real discovery call. Don't short-cut.
- Scope memo in under a week is the real product promise — here, it's same-day.
- Build on your stack. No net-new infra for the MVP.
- Every agent ships with a kill switch, scoped creds, and an audit log. No exceptions.
- If you finish early, stress the audit trail, not the happy path.
