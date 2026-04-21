# Scope memo — Creator-tech brand safety

**Prospect (simulated):** [Influencer marketing platform, 150 brands, 12K creators]
**Date:** [ ]
**Civic team:** Brad (SME), Titus (GTM), [engineer TBD]
**Tier:** Standard/Platform ($25K base, 60 days; platform if templated across brands)

## The agent

**Name it in one line.** [e.g. "Creator content brand-safety scorer"]

**What it does.** [Ingests creator content (text, image, video transcript), loads brand-specific safety rules, scores content against each rule, produces approve/flag/reject decision with reasoning, generates brand-ready compliance report.]

**What it does not do.** [Does not publish or schedule content. Does not contact creators. Does not make final brand-side approval decisions.]

## Success criteria

| Metric | Today | Pilot target |
|---|---|---|
| Review time per piece of content | | |
| Content reviewed per analyst per day | | |
| Consistency rate (same content = same score) | | |
| False-negative rate (missed violations) | | |
| Time to onboard new brand rules | | 1 day |

## Inputs

- [ ] Creator content — post text, image, video transcript (or screenshot)
- [ ] Brand safety ruleset per brand (YAML config)
- [ ] Creator profile — recent posts, audience demographics, past violations
- [ ] Platform content taxonomy (severity tiers, category definitions)
- [ ] Prior review decisions (calibration — at least 50 per brand)

## System of record

- **Agent writes to:** [Review queue — scored content staged for analyst/brand approval]
- **Audit log:** [Every content piece scored, rules applied, signals detected, decision + reasoning — timestamped]
- **Credentials scoped to:** [Read-only on content and creator profiles. No write to CMS or brand portals.]

## Governance envelope

- **Kill switch:** VP Brand Partnerships + platform ops lead
- **Human-in-loop triggers:**
  - Confidence below threshold (e.g. <0.85)
  - Regulated product category (alcohol, pharma, financial services)
  - Political or social-issue adjacency
  - Creator with prior violation history
  - New brand with <50 calibration examples
- **Audit:** Every score links to the content analyzed, rules matched, signals detected, and reasoning. Brand asks "why approved?" — answer in seconds.

## Team assigned

- Domain SME: Brad
- Lead engineer: [ ]
- GTM sponsor: Titus

## Replication

Brand-safety ruleset per brand + shared severity taxonomy. Adding a new brand = load their ruleset + 50 calibration examples + set thresholds. Target: 1 day.

## Open questions

- [ ]
- [ ]
