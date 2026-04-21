# NationalBrand co-op ad program rules

Version 2026.1. Applies to all franchisees submitting ads for co-op reimbursement.

## Brand guideline rules

| Rule ID | Rule | Severity |
|---|---|---|
| brand.logo.clear_space | Logo must have clear space of at least 1x logo height on all sides | blocker |
| brand.logo.color | Logo must be brand red (#C8102E) or white on dark bg | blocker |
| brand.colors.palette | Only brand palette colors permitted. No off-brand colors as primary | blocker |
| brand.typography.font | Headlines must use "NationalSans". Body must use "NationalSerif" | warning |
| brand.tone.language | No slang, profanity, or inflammatory language | blocker |
| brand.imagery.faces | Faces must be on pre-approved model list | blocker |
| brand.imagery.competitors | No competitor logos, products, or trademarks | blocker |

## Co-op program rules

| Rule ID | Rule | Severity |
|---|---|---|
| coop.claims.pricing | Pricing claims must cite date and disclaimer | blocker |
| coop.claims.health | No health or safety claims under any condition | blocker |
| coop.claims.limited_time | "Limited time" must include end date | warning |
| coop.local.location | Must include franchisee's location or phone | blocker |
| coop.local.spend_cap | Ad spend must be under $10K without HQ approval | blocker |
| coop.placement.platform | Permitted: Meta, Google, local print. Not permitted: TikTok, X, LinkedIn without HQ approval | warning |

## Human-in-loop triggers (hard)

- Any blocker violation.
- Any claim about pricing, health, safety.
- Any face not on pre-approved list.
- Any competitor reference.
- Agent confidence below 0.80.

## Pre-approved model list

model_001, model_002, model_003, model_004, model_005.
