# Sample inputs — CAS close + AP

This folder contains:

- `portco_001_coa.csv` — chart of accounts, Acme Manufacturing
- `portco_001_vendors.csv` — known vendor list with historical GL
- `portco_001.yaml` — portco config
- `portco_002_coa.csv` — chart of accounts, BetaServices LLC (different structure)
- `portco_002_vendors.csv` — different vendor set
- `portco_002.yaml` — portco config
- `invoices.jsonl` — 15 sample invoices as structured JSON (use these instead of real PDFs for the hackathon — saves you building a PDF extractor)
- `expected_outcomes.json` — ground truth for each invoice

## Why JSON instead of real PDFs

For the 2-day hackathon, use structured JSON. If a team finishes early, they can swap in real PDFs as a stretch.
