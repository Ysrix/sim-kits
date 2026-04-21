# Demo script — Agency client reporting agent

**Length:** 7 min + 3 min Q&A
**Presenters:** Engineer lead + Titus

## Flow

### 1. Scope recap (45s)
Hours per report today, hours target. Client count. Anomaly expectations.

### 2. Clean month (90s)
Run Acme DTC. Show:
- Config loaded, data pulled
- KPIs computed
- Narrative drafted
- Pull log traces every number

### 3. The anomaly (120s)
Run Beta SaaS. CAC up 32%. Agent flags with reasoning: "CAC $62 vs. $47 prior month (+32%). Meta CPMs up 28% in this client's category per DSP data, driving most of the delta."
Show:
- Anomaly block in report
- Reasoning cites source data, not guesses
- AM knows exactly what to validate before sending

### 4. The data gap (90s)
Run Gamma Brand. Meta data missing days 14–18. Agent does NOT interpolate. Produces partial report with explicit gap notice. Shows which days are missing, recommends AM pulls manually before final send.

### 5. The audit trace (60s)
Click on "ROAS 3.4x" in the Acme report. Show it resolves to the exact pull log entry. "Pulled from Google Ads API at 09:02 UTC. Query covered 2026-04-01 to 2026-04-30. Returned 8,412 rows. Sum of conversions_value / sum of spend."

### 6. Adding a new client (45s)
Drop in a fourth client config. Run it. No code changes. This is the replication proof.

### 7. What the prospect buys (30s)
Titus: $25K, 60 days, 30 clients covered from day one, 1-day onboarding for new clients, full audit trace on every number.

## Q&A prep

- **"What about Supermetrics/Funnel/Whatagraph?"** Those aggregate. They don't write the narrative, don't flag anomalies with reasoning, don't trace back to the pull. Agent sits on top — can read from them if you already have them.
- **"Will the agent hallucinate numbers?"** Numbers come from the pull log, not the model. Model writes narrative given the numbers. If a number is in the report, it came from a logged pull.
- **"What if the client disputes a number?"** Pull log gives you the source, time, query. You can answer in seconds.
- **"How does the agent know our brand voice?"** 3 months of prior reports as calibration. It matches tone and structure.

## Don't
- Don't let the agent invent numbers in the demo. Always show the pull log trace.
- Don't skip the data-gap case. "It fails safely" is the most convincing slide.
