# Claude Code scaffold — CAS close + AP agent

## Setup

```bash
cd ~/civic-sim/02-cas-close-pe
pip install anthropic pypdf pyyaml
export ANTHROPIC_API_KEY=sk-ant-...
```

## Directory layout

```
02-cas-close-pe/
├── agent.py
├── inputs/
│   ├── portco_001.yaml          # portco config
│   ├── portco_001_coa.csv       # chart of accounts
│   ├── portco_001_vendors.csv   # known vendors + historical GL
│   ├── portco_002.yaml
│   ├── portco_002_coa.csv
│   ├── portco_002_vendors.csv
│   ├── invoices.jsonl           # 15 sample invoices (structured JSON, not PDFs)
│   └── expected_outcomes.json   # ground truth for validation
├── config/
│   └── kill_switch.flag         # create at runtime to halt
└── outputs/
    ├── audit_log.jsonl
    ├── staged_entries/
    └── approval_queue/
```

> **Note:** For the hackathon, invoices are structured JSONL (pre-extracted).
> PDF ingestion via pypdf/haiku is a stretch goal — swap the loader, keep the coding logic.

## System prompt

```
You are an AP coding agent for a PE-backed accounting rollup. You code vendor invoices to the portfolio company's general ledger according to:

1. The portco's chart of accounts (provided)
2. Prior coding history (provided)
3. Approval rules (provided)

For each invoice, you will:
1. Extract vendor, amount, dates, line items
2. Propose GL coding with confidence score
3. Apply approval routing rules
4. Stage a journal entry (do NOT post)
5. Log your reasoning for audit

You NEVER:
- Invent GL codes not in the chart
- Post directly to the GL
- Bypass approval rules
- Auto-approve an invoice from a vendor with no prior history

You ALWAYS flag for human review when:
- Extraction confidence below 0.85
- First invoice from a vendor in 180 days
- Amount above portco threshold
- GL category differs from vendor's historical pattern

Return JSON only in the specified schema.
```

## Core loop

```python
import anthropic, json, yaml, pathlib, csv, base64
from datetime import datetime
from pypdf import PdfReader

client = anthropic.Anthropic()
KILL = pathlib.Path("config/kill_switch.flag")

def load_portco_config(portco_id):
    with open(f"inputs/{portco_id}.yaml") as f:
        cfg = yaml.safe_load(f)
    with open(cfg["chart_of_accounts"]) as f:
        cfg["coa"] = list(csv.DictReader(f))
    with open(cfg["known_vendors"]) as f:
        cfg["vendors"] = list(csv.DictReader(f))
    return cfg

def load_invoices(portco_id):
    """Load pre-extracted invoices from JSONL. For PDF stretch goal, swap this with extract_invoice()."""
    invoices = []
    with open("inputs/invoices.jsonl") as f:
        for line in f:
            inv = json.loads(line)
            if inv["portco_id"] == portco_id:
                invoices.append(inv)
    return invoices

def code_and_route(invoice, portco_cfg):
    coa_text = "\n".join([f"{a['code']} - {a['name']}" for a in portco_cfg["coa"]])
    vendors_text = "\n".join([f"{v['name']}: typical GL {v.get('typical_gl', 'unknown')}" for v in portco_cfg["vendors"]])
    rules_text = json.dumps(portco_cfg["approval_rules"], indent=2)

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2000,
        system=SYSTEM_PROMPT,
        messages=[{
            "role": "user",
            "content": f"""Invoice:
{json.dumps(invoice, indent=2)}

Chart of accounts:
{coa_text}

Known vendors and their typical coding:
{vendors_text}

Approval rules:
{rules_text}

Return a staged journal entry JSON with: entry_id, debit (account + amount), credit (account + amount), status, approver, agent_reasoning, human_review_required, human_review_reason."""
        }]
    )
    return json.loads(response.content[0].text)

def process_invoice(invoice, portco_id):
    if KILL.exists():
        return {"status": "halted", "reason": "kill_switch"}

    cfg = load_portco_config(portco_id)
    entry = code_and_route(invoice, cfg)

    # Audit
    audit = {
        "timestamp": datetime.utcnow().isoformat(),
        "portco_id": portco_id,
        "invoice_id": invoice.get("invoice_id"),
        "invoice": invoice,
        "entry": entry
    }
    with open("outputs/audit_log.jsonl", "a") as f:
        f.write(json.dumps(audit) + "\n")

    # Stage or queue
    if entry["human_review_required"]:
        queue_path = f"outputs/approval_queue/{entry['entry_id']}.json"
    else:
        queue_path = f"outputs/staged_entries/{entry['entry_id']}.json"
    pathlib.Path(queue_path).parent.mkdir(parents=True, exist_ok=True)
    with open(queue_path, "w") as f:
        json.dump(entry, f, indent=2)

    return entry

if __name__ == "__main__":
    for portco in ["portco_001", "portco_002"]:
        invoices = load_invoices(portco)
        for inv in invoices:
            print(process_invoice(inv, portco))
```

## What to build first

1. Load configs for both portcos.
2. Load one invoice from JSONL.
3. Code + stage a single entry.
4. Add audit log.
5. Add approval routing.
6. Run all 15 invoices across both portcos.
7. Prove portco swap works by running portco_002 without code changes.

## Debugging tips

- If GL coding is wrong, check that `vendors.csv` has the vendor's historical GL. Model relies on pattern-matching to the known list.
- If confidence is miscalibrated, add 3–5 worked examples to the system prompt.
- Use QBO/Xero sandbox APIs in stretch mode — do not touch live GLs during the hackathon.
