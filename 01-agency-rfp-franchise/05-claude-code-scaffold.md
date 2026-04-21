# Claude Code scaffold — Agency RFP + franchise co-op compliance

## Setup

```bash
cd ~/civic-sim/01-agency-rfp-franchise
npm init -y
npm install @anthropic-ai/sdk
# or: pip install anthropic
export ANTHROPIC_API_KEY=sk-ant-...
```

## Directory layout

```
01-agency-rfp-franchise/
├── agent.py                # or agent.js
├── rules/
│   ├── brand_rules.md      # brand + co-op rules with rule IDs
│   └── coop_rules.md       # (or merged into brand_rules.md)
├── inputs/
│   └── submissions.json    # array of 10 submissions with image_description fields
├── outputs/
│   ├── audit_log.jsonl
│   └── decisions/
└── config/
    ├── human_in_loop_rules.json
    └── kill_switch.flag
```

> **Note:** For the hackathon, submissions use `image_description` text fields instead of
> actual image files. The agent scores based on the description. Swap in real images
> (base64 via the Vision API) as a stretch goal.

## System prompt

```
You are a co-op ad compliance reviewer for [FRANCHISOR_NAME]. Your job is to decide whether a franchisee's submitted ad meets brand guidelines and co-op program rules.

You will receive:
- An ad image
- Ad copy text
- Submission metadata (franchisee ID, location, spend)
- The brand guidelines (attached)
- The co-op program rules (attached)

Return a decision in the exact JSON schema provided. Cite specific rule IDs. If confidence is below 0.80, or if ANY of the human-in-loop triggers fire, set human_review_required: true.

Human-in-loop triggers (HARD):
- Third-party trademark use
- Pricing, health, or safety claims
- Faces not on pre-approved list
- Any rule you are uncertain about

Never approve without citing the rules checked. Never invent rules not in the attached documents. If the input is malformed, return a parse_error decision with human_review_required: true.

Every decision is logged and audited.
```

## Core loop (Python pseudocode)

```python
import anthropic, json, pathlib
from datetime import datetime

client = anthropic.Anthropic()
KILL_SWITCH = pathlib.Path("config/kill_switch.flag")
AUDIT_LOG = pathlib.Path("outputs/audit_log.jsonl")

def is_killed():
    return KILL_SWITCH.exists()

def load_rules():
    brand_md = open("rules/brand_rules.md").read()
    return brand_md

def review_submission(submission):
    if is_killed():
        return {"decision": "halted", "reason": "kill_switch_active"}

    brand_rules = load_rules()

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2000,
        system=SYSTEM_PROMPT,
        messages=[{
            "role": "user",
            "content": [
                {"type": "text", "text": f"Brand and co-op rules:\n{brand_rules}"},
                {"type": "text", "text": f"Submission (ad copy + image description):\n{json.dumps(submission, indent=2)}\n\nReturn JSON only."}
            ]
        }]
    )

    decision = json.loads(response.content[0].text)
    audit_entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "submission_id": submission["id"],
        "decision": decision,
        "model": "claude-sonnet-4-6",
        "kill_switch_state": False
    }
    AUDIT_LOG.parent.mkdir(exist_ok=True)
    with AUDIT_LOG.open("a") as f:
        f.write(json.dumps(audit_entry) + "\n")
    return decision

if __name__ == "__main__":
    submissions = json.load(open("inputs/submissions.json"))
    for sub in submissions:
        print(review_submission(sub))
```

> **Stretch:** To use actual images instead of descriptions, add image files to `inputs/`,
> reference them via an `image_path` field, and send base64-encoded content blocks
> alongside the text. The scoring logic stays the same.

## What to build first

1. Mock brand guidelines + co-op rules (use sample inputs).
2. Get one submission through end-to-end.
3. Add audit log.
4. Add kill switch.
5. Run all 10 samples.
6. Stretch: replication swap.

## Debugging tips

- If decisions are inconsistent, lower temperature to 0 and re-test.
- If confidence is always high, your prompt isn't asking for calibration. Add: "State your confidence on a 0-1 scale based on how clearly the rules apply."
- If the model is citing rules that don't exist, tighten the system prompt: "Only cite rule IDs that appear verbatim in the attached documents."
