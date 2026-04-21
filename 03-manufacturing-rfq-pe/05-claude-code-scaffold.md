# Claude Code scaffold — Manufacturing RFQ agent

## Setup

```bash
cd ~/civic-sim/03-manufacturing-rfq-pe
pip install anthropic pyyaml
export ANTHROPIC_API_KEY=sk-ant-...
```

## Directory layout

```
03-manufacturing-rfq-pe/
├── agent.py
├── config/
│   ├── portco_001.yaml
│   ├── portco_002.yaml
│   └── kill_switch.flag
├── inputs/
│   ├── rfqs/
│   │   ├── rfq_001.json
│   │   └── ...
│   ├── portco_001_catalog.csv
│   ├── portco_001_inventory.json
│   ├── portco_001_customers.csv
│   ├── portco_001_pricing.yaml
│   ├── portco_002_catalog.csv
│   ├── portco_002_inventory.json
│   ├── portco_002_customers.csv
│   └── portco_002_pricing.yaml
└── outputs/
    ├── audit_log.jsonl
    └── quotes/
```

## System prompt

```python
SYSTEM_PROMPT = """You are an RFQ response drafter for a mid-market specialty manufacturer. You parse incoming RFQs and draft quote responses based on:

1. The portco's product catalog (provided)
2. Current inventory and lead times (provided)
3. Pricing rules and customer tier (provided)
4. Margin floor by product family (provided)

For each RFQ, you:
1. Parse line items — requested spec, quantity, needed-by date
2. Match each item to the catalog (requires confidence >= 0.85)
3. Check inventory and lead time
4. Apply pricing rules based on customer tier
5. Verify margin is at or above floor for every line
6. Draft a quote with line-by-line reasoning

You NEVER:
- Match a spec below 0.85 confidence without flagging
- Price below margin floor
- Quote a new customer without flagging
- Send the quote — you only draft

You ALWAYS flag for human review when:
- Any line match confidence below 0.85
- Any line margin below floor
- Non-standard spec that doesn't match the catalog
- New customer (not in customer_tier file)
- Quote total above portco auto-draft threshold

Return JSON only in the specified schema with agent_reasoning explaining every decision."""
```

## Core loop

```python
import anthropic, json, yaml, csv, pathlib
from datetime import datetime

SYSTEM_PROMPT = """You are an RFQ response drafter for a mid-market specialty manufacturer. You parse incoming RFQs and draft quote responses based on:

1. The portco's product catalog (provided)
2. Current inventory and lead times (provided)
3. Pricing rules and customer tier (provided)
4. Margin floor by product family (provided)

For each RFQ, you:
1. Parse line items — requested spec, quantity, needed-by date
2. Match each item to the catalog (requires confidence >= 0.85)
3. Check inventory and lead time
4. Apply pricing rules based on customer tier
5. Verify margin is at or above floor for every line
6. Draft a quote with line-by-line reasoning

You NEVER:
- Match a spec below 0.85 confidence without flagging
- Price below margin floor
- Quote a new customer without flagging
- Send the quote — you only draft

You ALWAYS flag for human review when:
- Any line match confidence below 0.85
- Any line margin below floor
- Non-standard spec that doesn't match the catalog
- New customer (not in customer_tier file)
- Quote total above portco auto-draft threshold

Return JSON only in the specified schema with agent_reasoning explaining every decision."""

client = anthropic.Anthropic()
KILL = pathlib.Path("config/kill_switch.flag")

def load_portco(portco_id):
    with open(f"config/{portco_id}.yaml") as f:
        cfg = yaml.safe_load(f)
    cfg["catalog"] = list(csv.DictReader(open(cfg["catalog_path"])))
    cfg["inventory"] = json.load(open(cfg["inventory_path"]))
    cfg["customers"] = {r["customer"]: r for r in csv.DictReader(open(cfg["customers_path"]))}
    with open(cfg["pricing_path"]) as f:
        cfg["pricing"] = yaml.safe_load(f)
    return cfg

def draft_quote(rfq, portco_cfg):
    if KILL.exists():
        return {"status": "halted"}

    prompt = f"""RFQ:
{json.dumps(rfq, indent=2)}

Catalog (first 50 items, full catalog available on request):
{json.dumps(portco_cfg["catalog"][:50], indent=2)}

Inventory snapshot:
{json.dumps(portco_cfg["inventory"], indent=2)}

Customer record:
{json.dumps(portco_cfg["customers"].get(rfq["customer"], {"tier": "unknown", "status": "NEW_CUSTOMER"}), indent=2)}

Pricing rules:
{yaml.dump(portco_cfg["pricing"])}

Draft a quote following the schema. If any condition requires human review, set human_review_required true and state the reason in agent_reasoning. Return JSON only."""

    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=3000,
        system=SYSTEM_PROMPT,
        messages=[{"role": "user", "content": prompt}]
    )
    quote = json.loads(response.content[0].text)

    audit = {
        "timestamp": datetime.utcnow().isoformat(),
        "rfq_id": rfq["rfq_id"],
        "portco_id": portco_cfg["portco_id"],
        "quote": quote
    }
    with open("outputs/audit_log.jsonl", "a") as f:
        f.write(json.dumps(audit) + "\n")

    out = pathlib.Path(f"outputs/quotes/{quote['quote_id']}.json")
    out.parent.mkdir(parents=True, exist_ok=True)
    out.write_text(json.dumps(quote, indent=2))
    return quote

def load_rfqs():
    """Load RFQs and strip test metadata (expected_outcome, expected_reason) before passing to model."""
    rfqs = json.load(open("inputs/rfqs/all_rfqs.json"))
    for rfq in rfqs:
        rfq.pop("expected_outcome", None)
        rfq.pop("expected_reason", None)
    return rfqs

if __name__ == "__main__":
    for rfq in load_rfqs():
        portco = load_portco(rfq["portco_id"])
        result = draft_quote(rfq, portco)
        print(result["quote_id"], "->", "REVIEW" if result.get("human_review_required") else "DRAFT")
```

## What to build first

1. Load portco 1 catalog, inventory, customers, pricing.
2. Get one RFQ through end-to-end.
3. Add margin check.
4. Add flag logic.
5. Add audit log.
6. Run all 10 RFQs.
7. Prove portco 2 swap.

## Debugging tips

- If catalog match is bad, try passing only top-20 candidates filtered by keyword match first, then let the model pick.
- If margin calculation drifts, put the margin formula in the prompt explicitly.
- If the model invents SKUs, tighten: "Only cite sku values that appear verbatim in the catalog list."
