# Build spec — Creator-tech brand safety

## MVP scope for Day 2

1. Load brand safety ruleset (YAML config per brand).
2. Load creator content submission (text + image description or transcript).
3. Load creator profile (recent posts summary, audience demo, violation history).
4. Score content against each rule in the brand's ruleset.
5. Produce aggregate safety score + per-rule breakdown.
6. Classify: approve, flag-for-review, reject — with reasoning per signal.
7. Generate brand-ready compliance report (Markdown for MVP).
8. Every scoring decision logged to audit trail.
9. Kill switch halts queue processing.

## Out of scope

- Live content ingestion from social APIs — mock data only.
- Video frame analysis — use transcript + description.
- Auto-publish or auto-reject (agent scores, human decides).
- Creator outreach or feedback messaging.
- Audience overlap analysis across brands.

## Stack

- Python 3.11.
- Claude API (sonnet-4-6 for content analysis + reasoning, haiku-4-5 for rule matching).
- Vision API for image scoring (sonnet-4-6 with image input).
- YAML configs per brand.
- Markdown + JSONL output.

## Architecture

```
brand_rules.yaml ──────────┐
                           ├──► [load context] ──► rules + creator profile
creator_profile.json ──────┤                              │
                           │                              ▼
content_submission.json ───┘                     [score content]
                                                          │
                                              ┌───────────┼───────────┐
                                              ▼           ▼           ▼
                                         [text rules] [image rules] [profile rules]
                                              │           │           │
                                              └───────────┼───────────┘
                                                          ▼
                                                  [aggregate score]
                                                          │
                                                          ├──► decision.json
                                                          ├──► compliance_report.md
                                                          └──► audit_log.jsonl
```

## Brand safety ruleset schema

```yaml
brand_id: outdoor_co
brand_name: "OutdoorCo"
severity_tiers:
  instant_reject:
    - hate_speech
    - explicit_sexual
    - illegal_activity
    - competitor_mention
  flag_for_review:
    - alcohol_visible
    - political_content
    - profanity_mild
    - controversial_topic
  acceptable:
    - outdoor_activity
    - fitness
    - family_content
    - travel

thresholds:
  min_approve_score: 0.85
  auto_reject_below: 0.30
  flag_band: [0.30, 0.85]

creator_history:
  violation_lookback_days: 180
  max_violations_before_block: 2

audience:
  min_brand_affinity_pct: 40    # % of audience in brand's target demo
  max_bot_pct: 15

rules:
  - id: R001
    name: "No competitor products"
    type: text_and_image
    keywords: ["CompetitorA", "CompetitorB", "RivalBrand"]
    severity: instant_reject
  - id: R002
    name: "No alcohol in primary frame"
    type: image
    signal: alcohol_visible
    severity: flag_for_review
  - id: R003
    name: "Brand tone alignment"
    type: text
    description: "Content tone must align with outdoors/adventure/family-friendly"
    severity: flag_for_review
  - id: R004
    name: "Audience quality"
    type: profile
    check: "bot_pct <= max_bot_pct AND brand_affinity_pct >= min_brand_affinity_pct"
    severity: flag_for_review
```

## Content submission schema

```json
{
  "submission_id": "SUB-20260417-001",
  "creator_id": "creator_042",
  "brand_id": "outdoor_co",
  "campaign_id": "CAMP-2026Q2-003",
  "platform": "instagram",
  "content_type": "carousel",
  "text": "Nothing beats a sunrise hike with the right gear. @OutdoorCo keeps me moving. #sponsored #outdoors",
  "image_descriptions": [
    "Person on mountain trail at sunrise wearing OutdoorCo jacket, mountain range in background",
    "Close-up of hiking boots on rocky terrain, OutdoorCo logo visible"
  ],
  "hashtags": ["sponsored", "outdoors", "hiking", "sunrise"],
  "submitted_at": "2026-04-17T10:30:00Z"
}
```

## Creator profile schema

```json
{
  "creator_id": "creator_042",
  "handle": "@trail_runner_jay",
  "platform": "instagram",
  "followers": 85000,
  "audience_demo": {
    "age_18_34_pct": 62,
    "brand_affinity_pct": 58,
    "bot_pct": 4
  },
  "recent_posts_summary": "Outdoor fitness, trail running, camping gear reviews. Occasionally posts about craft beer.",
  "violation_history": [],
  "past_brand_partners": ["GearCo", "TrailMix Inc"]
}
```

## Scoring output schema

```json
{
  "submission_id": "SUB-20260417-001",
  "brand_id": "outdoor_co",
  "overall_score": 0.92,
  "decision": "approve",
  "rule_results": [
    {"rule_id": "R001", "passed": true, "detail": "No competitor mentions found"},
    {"rule_id": "R002", "passed": true, "detail": "No alcohol detected in image descriptions"},
    {"rule_id": "R003", "passed": true, "detail": "Tone aligns with outdoor/adventure positioning"},
    {"rule_id": "R004", "passed": true, "detail": "Bot 4% (≤15%), affinity 58% (≥40%)"}
  ],
  "reasoning": "Content is on-brand: outdoor activity, proper disclosure (#sponsored), no competing products, clean creator history. Audience quality within thresholds.",
  "flags": []
}
```

## Human-in-loop triggers

- Overall score in flag band (0.30–0.85)
- Any instant_reject signal detected (human confirms before final reject)
- Creator has prior violations within lookback window
- Regulated product adjacency (alcohol, pharma, financial)
- New brand with <50 calibration examples
- Ambiguous image content (model confidence <0.70 on any image signal)

## Success check for demo

8 content submissions in `inputs/`:
- 3 clean approves (high score, no flags)
- 2 flag-for-review (one alcohol adjacency, one audience quality borderline)
- 1 instant reject (competitor mention in image)
- 1 creator with violation history (otherwise clean content — tests history check)
- 1 second-brand submission (proves config swap / replication)

Agent must:
- Correctly score and classify all 8
- Produce per-rule breakdown for each
- Generate compliance report with reasoning
- Handle the brand-config swap without code changes
- Produce audit log

## Stretch

- Video transcript analysis (YouTube/TikTok long-form)
- Batch scoring with priority queue (urgent campaigns first)
- Brand-side dashboard with drill-down per rule
- Creator risk score (aggregate across all submissions)
