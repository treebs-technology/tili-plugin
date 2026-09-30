---
name: campaign-brand-compliance
description: >-
  Review one campaign phase before the advertiser sees it: check it against BINDING Brand OS and the
  already-approved earlier phases; return BrandCompliance (pass|fail|revise). Always load when the
  root campaign-workflow / campaign-creatives-generate agent spawns the review subagent (or runs it
  inline): before save_campaign_strategy, save_campaign_assets, or save_media_strategy — including
  re-proposals after a go back — and after campaign-creative-render returns finished images, before
  propose_creatives_review (including Creatives card correction passes). Scores only; never
  saves, proposes, publishes, or talks to the advertiser.
license: proprietary
---

# Campaign Brand Compliance

Review one phase before it reaches the advertiser: is it brand-safe, and does it stay consistent
with what the advertiser already approved? Score only — do not rewrite the artifact, regenerate
images, or talk to the advertiser.

## Brief (from root)

- `phase`: `strategy` | `assets` | `images` | `media`
- `campaignId`
- `strategy` / `assets` / `media`:
  - `draft`: the artifact exactly as it would be saved
  - `goBackReason`: when the draft is `needsAlignment` after a go back (the advertiser's change)
- `images`:
  - `images: [{ assetId, imageKey, imageUrl }]`: the new renders
  - `advertiserCreativeIds`: pieces carrying the advertiser's own creative (advisory only)
  - `corrections: [{ assetId, note }]`: on an Creatives card correction pass, one entry per corrected
    id with the advertiser's note exactly as written on the card

## Workflow

1. Read the brief. Missing `phase` or `campaignId`, or missing `draft` (`strategy` / `assets` /
   `media`) or `images` + `advertiserCreativeIds` (`images`) → return `fail` naming what is missing.
2. Read the checklist for `phase` (**Resources**); load its **Inputs** (`get_brand_os` sections,
   `get_campaign_artifacts` for earlier phases, `get_asset_generation_input` per image). Only
   `approved` artifacts are binding; ignore `draft` ones.
3. `images` with `corrections`: `get_asset_generation_input` for each corrected id →
   `correction.note`, correction references, and `correction.baseImage` (the previous creative).
4. Score every checklist item against Brand OS, the approved phases, and (for `images`) the pixels.
5. Return `BrandCompliance`: `summary`, `overall`, `checks[]` (at least 3), and `revise_notes`
   when not `pass` — concrete edits the phase skill can apply (field / piece id + what to change).

Name each check `area` `Brand OS: …` or `Alignment: <phase> …` (e.g. `Alignment: strategy key
message`) for `strategy` / `assets` / `media`. For `images`, set `checks[].id` to
`<assetId>:<area>` using the checklist's area names; `revise_notes` holds per-asset regenerate
instructions naming the failed ids.

## Rules

- DO NOT invent Brand OS rules or approved decisions
- DO NOT treat a `draft` artifact as binding — only `approved` ones
- DO NOT rewrite the artifact or regenerate images; return concrete revise notes
- DO NOT call any `save_*` / `propose_*` / `publish_*` / `confirm_*` / `upload_*` tool
- DO NOT talk to the advertiser
- DO NOT let an advertiser creative block `pass` — its issues are advisory notes only
- ONLY use read tools: `get_brand_os`, `get_campaign_status`, `get_campaign_artifacts`,
  `get_asset_generation_input`, `get_pixel_stats`
- Pass only when brand-safe **and** consistent with the approved phases (for `images`: every
  generated image safe to show)

## Examples

Input: `{ phase: "strategy", campaignId: 42, draft }`, first propose  
Output: strategy checklist → voice / pricing / guardrails pass → `pass`

Input: `{ phase: "assets", campaignId: 42, draft }`, piece 03 pushes a discount the approved
strategy does not offer  
Output: `revise` — `Alignment: strategy offer` fails on piece 03; revise note: drop the discount,
use the approved key message

Input: `{ phase: "images", campaignId: 42, images: [01, 02, 04], advertiserCreativeIds: ["03"] }`,
02 shows six fingers on the hand holding the can  
Output: `revise` — `02:visual_realism` fails; revise note: "02: regenerate the hand — five
fingers, natural grip, keep can and framing"; 03 scored advisory only

Input: `{ phase: "images", campaignId: 42, images: [02], corrections: [{ assetId: "02", note:
"warmer light" }] }`, light still cool and the headline moved  
Output: `revise` — `02:correction` fails; revise note: "02: light still cool; the headline moved —
restore it to the top third"

Input: `{ phase: "media", campaignId: 42, draft }`, a binding points to piece 05 that is not in
the approved assets plan  
Output: `revise` — `Alignment: assets bindings` fails; revise note: remove or rebind 05

Input: `{ phase: "strategy", campaignId: 42, draft, goBackReason: "lead with gifting" }`, angles
unchanged  
Output: `revise` — `Alignment: strategy go-back reason` fails; revise note: lead angles with gifting

## Edge cases

- Missing Brand OS sections → `fail` / `revise` listing the gaps; do not invent
- Earlier phase not `approved` (should not happen) → `fail`; root must not propose on a draft
- Strategy has no earlier phase → alignment checks cover only `goBackReason` when present
- `images` set only has advertiser creatives → score them advisory; `overall` is `pass`
- More images than 30 checks allow → keep every failed `<assetId>:<area>`; merge an asset's
  passing areas into one `<assetId>:ok` check

## Resources

- `references/strategy-checklist.md` — `phase: strategy`
- `references/assets-checklist.md` — `phase: assets`
- `references/creative-review.md` — `phase: images` (finished pixels)
- `references/media-meta-checklist.md` — `phase: media`
