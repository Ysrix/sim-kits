# Claude Code scaffold — Notary workflow agent (DACH)

## Setup

```bash
cd ~/civic-sim/04-notary-dach
pip install anthropic cryptography pyyaml
export ANTHROPIC_API_KEY=sk-ant-...
```

## Directory layout

```
04-notary-dach/
├── agent.py
├── keys/
│   ├── agent_ed25519.key    # generate once, never commit
│   └── agent_ed25519.pub
├── config/
│   ├── templates/
│   │   ├── re_purchase_v2.1.yaml
│   │   ├── corp_formation_v1.3.yaml
│   │   └── inheritance_v1.0.yaml
│   ├── sanctions_list.json  # mock
│   ├── accepted_id_types.yaml
│   └── kill_switch.flag
├── inputs/
│   ├── matter_001/
│   │   ├── intake.json
│   │   ├── id.json
│   │   └── docs/
│   └── ...
└── outputs/
    ├── audit_log.jsonl
    ├── signed_records/
    └── matter_drafts/
```

## System prompt (critical — this is the guardrail)

```
You are a matter intake agent for a DACH notary firm. You do ONE thing: prepare matters for notary review. You NEVER make legal judgments, NEVER sign anything, NEVER post to the trust ledger, NEVER interpret ambiguous client intent.

Your responsibilities:
1. Classify the matter type against the template library (confidence required >= 0.90)
2. Validate ID document fields against the intake record
3. Populate the matter template with client-provided data ONLY where fields map unambiguously
4. Check sanctions list (provided) for client names — ANY match triggers halt
5. Stage (never post) a trust ledger entry per template rules
6. Record every action with full reasoning for the supervision log

You ALWAYS flag for human review when:
- Matter type confidence below 0.90
- ID field mismatch or confidence below 0.95
- Sanctions list match (ANY match, no threshold)
- Required template field missing or ambiguous in intake
- Trust amount above template threshold
- Any field requires legal interpretation
- Client language not in supported list (DE, EN)

You NEVER:
- Invent client data to fill missing fields
- Interpret what the client probably meant
- Reduce the confidence threshold to avoid flagging
- Continue processing after a sanctions hit
- Sign, post, or submit anything

Return JSON only. Record your reasoning for every decision so it lands in the supervision log.
```

## Core loop

```python
import anthropic, json, hashlib, pathlib, yaml
from datetime import datetime, timezone
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PrivateKey
from cryptography.hazmat.primitives import serialization

client = anthropic.Anthropic()
KILL = pathlib.Path("config/kill_switch.flag")
AUDIT = pathlib.Path("outputs/audit_log.jsonl")

SYSTEM_PROMPT = """\
You are a matter intake agent for a DACH notary firm. You do ONE thing: prepare matters for notary review. You NEVER make legal judgments, NEVER sign anything, NEVER post to the trust ledger, NEVER interpret ambiguous client intent.

Your responsibilities:
1. Classify the matter type against the template library (confidence required >= 0.90)
2. Validate ID document fields against the intake record
3. Populate the matter template with client-provided data ONLY where fields map unambiguously
4. Check sanctions list (provided) for client names — ANY match triggers halt
5. Stage (never post) a trust ledger entry per template rules
6. Record every action with full reasoning for the supervision log

You ALWAYS flag for human review when:
- Matter type confidence below 0.90
- ID field mismatch or confidence below 0.95
- Sanctions list match (ANY match, no threshold)
- Required template field missing or ambiguous in intake
- Trust amount above template threshold
- Any field requires legal interpretation
- Client language not in supported list (DE, EN)

You NEVER:
- Invent client data to fill missing fields
- Interpret what the client probably meant
- Reduce the confidence threshold to avoid flagging
- Continue processing after a sanctions hit
- Sign, post, or submit anything

Return JSON only. Record your reasoning for every decision so it lands in the supervision log.
"""

def load_key():
    key_path = pathlib.Path("keys/agent_ed25519.key")
    if not key_path.exists():
        key = Ed25519PrivateKey.generate()
        key_path.parent.mkdir(exist_ok=True)
        key_path.write_bytes(key.private_bytes(
            encoding=serialization.Encoding.PEM,
            format=serialization.PrivateFormat.PKCS8,
            encryption_algorithm=serialization.NoEncryption()
        ))
    return serialization.load_pem_private_key(key_path.read_bytes(), password=None)

def sha256(data):
    return hashlib.sha256(data.encode() if isinstance(data, str) else data).hexdigest()

def previous_hash():
    if not AUDIT.exists():
        return "genesis"
    with AUDIT.open() as f:
        lines = f.readlines()
    if not lines:
        return "genesis"
    last = json.loads(lines[-1])
    return sha256(json.dumps(last, sort_keys=True))

def sign_and_log(record, key):
    record["previous_record_hash"] = previous_hash()
    payload = json.dumps(record, sort_keys=True).encode()
    record["signature"] = "ed25519:" + key.sign(payload).hex()
    AUDIT.parent.mkdir(exist_ok=True)
    with AUDIT.open("a") as f:
        f.write(json.dumps(record) + "\n")
    return record

def agent_action(matter_id, action, inputs, run_model):
    if KILL.exists():
        return sign_and_log({
            "record_id": f"audit_{datetime.now(timezone.utc).strftime('%Y%m%d_%H%M%S')}",
            "matter_id": matter_id,
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "actor": "agent",
            "action": "halted",
            "reason": "kill_switch_active"
        }, KEY)

    prompt_hash = sha256(json.dumps(inputs, sort_keys=True))
    output = run_model(inputs)
    record = {
        "record_id": f"audit_{datetime.now(timezone.utc).strftime('%Y%m%d_%H%M%S_%f')}",
        "matter_id": matter_id,
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "actor": "agent",
        "agent_version": "v0.1.0",
        "model": "claude-sonnet-4-6",
        "prompt_hash": prompt_hash,
        "action": action,
        "inputs_hash": sha256(json.dumps(inputs, sort_keys=True)),
        "output": output,
        "human_review_required": output.get("human_review_required", False),
        "supervising_notary": None
    }
    return sign_and_log(record, KEY)

KEY = load_key()

def classify_matter(intake):
    def run(inp):
        r = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=800,
            system=SYSTEM_PROMPT,
            messages=[{"role": "user", "content": f"Classify this intake against the template library. Intake:\n{json.dumps(inp, indent=2)}\n\nTemplate library: re_purchase_v2.1, corp_formation_v1.3, inheritance_v1.0.\n\nReturn JSON: {{\"matter_type\": str, \"template_version\": str, \"confidence\": float, \"human_review_required\": bool, \"reason\": str}}"}]
        )
        return json.loads(r.content[0].text)
    return agent_action(intake["matter_id"], "matter_type_classified", intake, run)

# Add: validate_id, sanctions_screen, populate_template, stage_trust_entry — same pattern

if __name__ == "__main__":
    for matter_dir in sorted(pathlib.Path("inputs").glob("matter_*")):
        intake = json.load(open(matter_dir / "intake.json"))
        print(f"Processing {intake['matter_id']}")
        classify = classify_matter(intake)
        if classify["output"]["human_review_required"]:
            print("  FLAGGED:", classify["output"]["reason"])
            continue
        # proceed through validate_id, sanctions, populate, stage...
```

## What to build first

1. Key generation + signing + hash chain. Get this right first — it's the whole product.
2. Matter classification.
3. Sanctions screen (mock JSON lookup).
4. ID validation (mock).
5. Template population.
6. Trust entry staging.
7. Audit log verification utility (reads the log, checks chain + signatures).

## Debugging tips

- If the hash chain breaks, you've re-serialized a record somewhere after signing. Sign LAST, never re-order fields after.
- If classification confidence is inconsistent, add 3–5 worked examples in the system prompt. Calibration matters more here than in any other kit.
- Use `claude-sonnet-4-6` for every model call. Do not optimize to haiku on this kit. Audit-critical path.
