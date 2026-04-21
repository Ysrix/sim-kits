# UrbanEats Franchising co-op ad program rules

Version 2026.1. Applies to all franchisees submitting ads for co-op reimbursement.

## Brand guideline rules

| Rule ID | Rule | Severity |
|---|---|---|
| brand.logo.clear_space | Logo must have clear space of at least 2x logo height on all sides | blocker |
| brand.logo.color | Logo must be forest green (#2D6A4F) or white on dark bg | blocker |
| brand.colors.palette | Only brand palette colors permitted: forest green (#2D6A4F), warm cream (#FAF3E0), charcoal (#333333). No off-brand colors as primary | blocker |
| brand.typography.font | Headlines must use "UrbanSlab". Body must use "UrbanSans" | warning |
| brand.tone.language | No slang, profanity, or inflammatory language. Tone must be warm and community-focused | blocker |
| brand.imagery.faces | Faces must be on pre-approved model list | blocker |
| brand.imagery.competitors | No competitor logos, products, or trademarks | blocker |
| brand.imagery.food | All food imagery must be from the approved photo library or shot by an approved photographer | blocker |

## Co-op program rules

| Rule ID | Rule | Severity |
|---|---|---|
| coop.claims.pricing | Pricing claims must cite date and disclaimer. No "lowest price" or "cheapest" language | blocker |
| coop.claims.health | No health, nutrition, or safety claims unless pre-approved by HQ legal | blocker |
| coop.claims.limited_time | "Limited time" must include end date and location | warning |
| coop.local.location | Must include franchisee's location, phone, and hours of operation | blocker |
| coop.local.spend_cap | Ad spend must be under $5K without HQ approval | blocker |
| coop.placement.platform | Permitted: Meta, Google, Yelp, local print. Not permitted: TikTok, X, Snapchat without HQ approval | warning |
| coop.delivery.disclaimer | Ads mentioning delivery must include "delivery fees may apply" disclaimer | blocker |

## Human-in-loop triggers (hard)

- Any blocker violation.
- Any claim about pricing, health, safety, or nutrition.
- Any face not on pre-approved list.
- Any competitor reference.
- Any food imagery not from approved library.
- Agent confidence below 0.85.

## Pre-approved model list

model_010, model_011, model_012, model_013.
