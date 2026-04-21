# Claude Code scaffold — Creator-tech brand safety

## Setup

```bash
cd ~/civic-sim/07-creator-brand-safety
pip install anthropic pyyaml
export ANTHROPIC_API_KEY=sk-ant-...
```

## Directory layout

```
07-creator-brand-safety/
├── agent.py
├── config/
│   ├── outdoor_co.yaml
│   ├── health_co.yaml
│   └── kill_switch.flag
├── inputs/
│   ├── submissions/
│   │   ├── sub_001.json
│   │   ├── sub_002.json
│   │   └── ...
│   ├── creators/
│   │   ├── creator_042.json
│   │   └── ...
│   └── expected_outcomes.json
└── outputs/
    ├── decisions/
    ├── compliance_reports/
    └── audit_log.jsonl
```

## System prompt

```
You are a brand-safety content analyst for an influencer marketing platform. You score creator content against a brand's safety ruleset and produce a structured decision.

Given:
1. The creator's content submission (text, image descriptions, hashtags)
2. The brand's safety ruleset (rules, severity tiers, thresholds)
3. The creator's profile (audience demographics, violation history, recent posts)

Your job:
- Evaluate each rule in the brand's ruleset against the content
- Score each rule as pass or fail with specific evidence
- Compute an aggregate safety score (0.0 to 1.0)
- Classify: approve (≥min_approve_score), flag_for_review (in flag_band), reject (<auto_reject_below)
- Write reasoning that a brand manager can read and act on

You NEVER:
- Approve content that triggers an instant_reject rule, regardless of overall score
- Ignore creator violation history
- Guess at image content beyond what the descriptions state
- Override the brand's threshold configuration

You ALWAYS:
- Evaluate EVERY rule in the ruleset — no skipping
- Cite the specific rule ID and name for each finding
- State the evidence (exact text match, image signal, profile metric)
- Flag ambiguity rather than guessing

Return JSON with: submission_id, brand_id, overall_score, decision, rule_results (array), reasoning, flags (array).
```

## Core loop

```python
import anthropic, json, yaml, pathlib
from datetime import datetime

SYSTEM_PROMPT = """You are a brand-safety content analyst for an influencer marketing platform. You score creator content against a brand's safety ruleset and produce a structured decision.

Given:
1. The creator's content submission (text, image descriptions, hashtags)
2. The brand's safety ruleset (rules, severity tiers, thresholds)
3. The creator's profile (audience demographics, violation history, recent posts)

Your job:
- Evaluate each rule in the brand's ruleset against the content
- Score each rule as pass or fail with specific evidence
- Compute an aggregate safety score (0.0 to 1.0)
- Classify: approve (>=min_approve_score), flag_for_review (in flag_band), reject (<auto_reject_below)
- Write reasoning that a brand manager can read and act on

You NEVER:
- Approve content that triggers an instant_reject rule, regardless of overall score
- Ignore creator violation history
- Guess at image content beyond what the descriptions state
- Override the brand's threshold configuration

You ALWAYS:
- Evaluate EVERY rule in the ruleset — no skipping
- Cite the specific rule ID and name for each finding
- State the evidence (exact text match, image signal, profile metric)
- Flag ambiguity rather than guessing

Return JSON with: submission_id, brand_id, overall_score, decision, rule_results (array), reasoning, flags (array)."""

client = anthropic.Anthropic()
KILL = pathlib.Path("config/kill_switch.flag")
AUDIT = pathlib.Path("outputs/audit_log.jsonl")

def log_audit(event_type, detail):
    entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "event": event_type,
        "detail": detail
    }
    AUDIT.parent.mkdir(exist_ok=True)
    with AUDIT.open("a") as f:
        f.write(json.dumps(entry) + "\n")

def load_brand_rules(brand_id):
    path = pathlib.Path(f"config/{brand_id}.yaml")
    return yaml.safe_load(open(path))

def load_submission(submission_id):
    path = pathlib.Path(f"inputs/submissions/{submission_id}.json")
    return json.load(open(path))

def load_creator(creator_id):
    path = pathlib.Path(f"inputs/creators/{creator_id}.json")
    return json.load(open(path))

def check_creator_history(creator, rules_cfg):
    """Check if creator has violations within lookback window."""
    lookback = rules_cfg.get("creator_history", {}).get("violation_lookback_days", 180)
    max_violations = rules_cfg.get("creator_history", {}).get("max_violations_before_block", 2)
    violations = creator.get("violation_history", [])
    # In production, filter by date. For MVP, count all.
    recent = len(violations)
    return {
        "has_violations": recent > 0,
        "violation_count": recent,
        "blocked": recent >= max_violations,
        "flag_for_review": 0 < recent < max_violations
    }

def check_audience(creator, rules_cfg):
    """Check audience quality metrics."""
    audience = creator.get("audience_demo", {})
    aud_rules = rules_cfg.get("audience", {})
    issues = []
    if audience.get("bot_pct", 0) > aud_rules.get("max_bot_pct", 15):
        issues.append(f"Bot % {audience['bot_pct']}% exceeds max {aud_rules['max_bot_pct']}%")
    if audience.get("brand_affinity_pct", 0) < aud_rules.get("min_brand_affinity_pct", 40):
        issues.append(f"Brand affinity {audience['brand_affinity_pct']}% below min {aud_rules['min_brand_affinity_pct']}%")
    return {"passed": len(issues) == 0, "issues": issues}

def score_content(submission, creator, rules_cfg):
    """Use Claude to score content against brand rules."""
    if KILL.exists():
        return {"status": "halted"}

    history_check = check_creator_history(creator, rules_cfg)
    audience_check = check_audience(creator, rules_cfg)

    prompt = f"""Score this content submission against the brand's safety rules.

Submission:
{json.dumps(submission, indent=2)}

Creator profile:
{json.dumps(creator, indent=2)}

Brand safety rules:
{json.dumps(rules_cfg, indent=2, default=str)}

Pre-computed checks:
- Creator history: {json.dumps(history_check)}
- Audience quality: {json.dumps(audience_check)}

Evaluate each rule. Return JSON with: submission_id, brand_id, overall_score, decision, rule_results, reasoning, flags."""

    r = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2500,
        system=SYSTEM_PROMPT,
        messages=[{"role": "user", "content": prompt}]
    )
    result = json.loads(r.content[0].text)

    # Apply hard overrides
    thresholds = rules_cfg.get("thresholds", {})

    # Instant reject override
    for rr in result.get("rule_results", []):
        rule_def = next((r for r in rules_cfg.get("rules", []) if r["id"] == rr.get("rule_id")), None)
        if rule_def and rule_def.get("severity") == "instant_reject" and not rr.get("passed"):
            result["overall_score"] = 0.0
            result["decision"] = "reject"
            result["flags"].append("instant_reject_signal")

    # Creator history override
    if history_check["flag_for_review"] and result["decision"] == "approve":
        result["decision"] = "flag_for_review"
        result["flags"].append("creator_violation_history")

    if history_check["blocked"]:
        result["decision"] = "reject"
        result["flags"].append("creator_blocked_violations")

    return result

def generate_compliance_report(result, submission, rules_cfg):
    """Generate a brand-facing compliance report."""
    report = f"# Brand Safety Review — {submission['submission_id']}\n\n"
    report += f"**Brand:** {rules_cfg.get('brand_name', result['brand_id'])}\n"
    report += f"**Creator:** {submission['creator_id']}\n"
    report += f"**Campaign:** {submission.get('campaign_id', 'N/A')}\n"
    report += f"**Platform:** {submission.get('platform', 'N/A')}\n"
    report += f"**Decision:** {result['decision'].upper()}\n"
    report += f"**Score:** {result['overall_score']}\n\n"
    report += "## Rule Results\n\n"
    for rr in result.get("rule_results", []):
        status = "PASS" if rr.get("passed") else "FAIL"
        report += f"- **{rr.get('rule_id', '?')}** ({status}): {rr.get('detail', '')}\n"
    report += f"\n## Reasoning\n\n{result.get('reasoning', '')}\n"
    if result.get("flags"):
        report += f"\n## Flags\n\n"
        for flag in result["flags"]:
            report += f"- {flag}\n"
    report += f"\n---\n*Generated {datetime.utcnow().isoformat()}Z*\n"
    return report

def run(submission_id):
    submission = load_submission(submission_id)
    brand_id = submission["brand_id"]
    creator_id = submission["creator_id"]

    rules_cfg = load_brand_rules(brand_id)
    creator = load_creator(creator_id)

    result = score_content(submission, creator, rules_cfg)

    # Write decision
    out_dec = pathlib.Path(f"outputs/decisions/{submission_id}.json")
    out_dec.parent.mkdir(parents=True, exist_ok=True)
    json.dump(result, open(out_dec, "w"), indent=2)

    # Write compliance report
    report = generate_compliance_report(result, submission, rules_cfg)
    out_rpt = pathlib.Path(f"outputs/compliance_reports/{submission_id}.md")
    out_rpt.parent.mkdir(parents=True, exist_ok=True)
    out_rpt.write_text(report)

    # Audit
    log_audit("content_scored", {
        "submission_id": submission_id,
        "brand_id": brand_id,
        "creator_id": creator_id,
        "score": result.get("overall_score"),
        "decision": result.get("decision"),
        "flags": result.get("flags", []),
        "model": "claude-sonnet-4-6"
    })

    return result

if __name__ == "__main__":
    submissions = ["sub_001", "sub_002", "sub_003", "sub_004",
                    "sub_005", "sub_006", "sub_007", "sub_008"]
    for sid in submissions:
        print(f"\n--- {sid} ---")
        print(json.dumps(run(sid), indent=2))
```

## What to build first

1. Load one brand config + one content submission.
2. Load creator profile.
3. Run content scoring against rules.
4. Apply hard overrides (instant_reject, creator history).
5. Generate compliance report.
6. Add audit log.
7. Test the flag-for-review path (alcohol adjacency).
8. Test the instant-reject path (competitor mention).
9. Test creator history override.
10. Run with second brand config (replication proof).

## Debugging tips

- If the model scores too generously, ensure the system prompt enforces "evaluate EVERY rule" — models tend to skip rules that seem obviously fine.
- If instant_reject content gets an approve score, check the hard-override logic runs AFTER the model returns — the model proposes, the code enforces.
- For image analysis, make sure image_descriptions in the submission are detailed enough. Vague descriptions produce vague scores.
- If audience checks seem wrong, verify the creator profile has the right field names (brand_affinity_pct, bot_pct) — typos here cause silent failures.
