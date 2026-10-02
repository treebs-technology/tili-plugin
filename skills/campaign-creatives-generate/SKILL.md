---
name: campaign-creatives-generate
description: >-
  Root-only orchestration of finished creative images AFTER G_ASSETS: fan out one
  campaign-creative-render subagent per planned piece in parallel, spawn the
  campaign-brand-compliance images review, open the Creatives card (hard gate G_CREATIVES,
  Approval 3/4); on approve the images join the workspace media library. Always load on the
  root / campaign-workflow agent when generating, reviewing, or correcting campaign creatives.
  Read references/creative-review.md before briefing the images review; read
  references/asset-metadata.md before propose_creatives_review.
license: proprietary
---

# Campaign Creatives Generate (root only)

Load on the **root / campaign-workflow** agent so the advertiser sees creatives in the root chat.
Prompts and references were fixed at G_ASSETS — this phase renders them, it does not re-plan or
look for new references.

## Workflow

1. Confirm G_ASSETS approved (status `PRODUCING_CREATIVES`). Read the approved plan
   (`get_campaign_artifacts` `artifact: "assets"`) and the images so far (`artifact:
   "creatives"`, one item per plan piece that has an image). Items with `source: "advertiser"`
   are the advertiser's own creative from the assets card — they are **not rendered**. Items
   that already have an `imageKey` are reused, **do not re-render** — the server keeps an image
   only while its piece's visual brief is unchanged (after a go back, changed pieces drop out on
   their own). Render only plan pieces without an image. Confirm product-image source only if at
   least one piece still needs rendering (ask only if missing).
2. **Fan out:** spawn one `campaign-creative-render` subagent per plan piece **without** an image,
   all in parallel. Brief each with `campaignId` + `assetId` (+ product-image source).
   Each renders from `get_asset_generation_input`, uploads with `upload_campaign_image`
   (`purpose: "creative"`), verifies the upload, and returns `{ assetId, imageKey, imageUrl }`.
3. **Images review:** spawn one `campaign-brand-compliance` subagent with `phase: "images"`,
   `campaignId`, the returned `{ assetId, imageKey, imageUrl }` set, and `advertiserCreativeIds`
   (the items with `source: "advertiser"`). On `revise`, re-spawn render subagents only for the
   failed generated ids, with the reviewer's `revise_notes` (max **2** revise rounds), then review
   those ids again. Advertiser creatives are never re-rendered — keep their review notes as
   advisory.
4. Metadata: read `references/asset-metadata.md` and write `metadata` for every image you send
   (view each finished image first).
5. `propose_creatives_review` with every newly generated `{ assetId, imageKey, metadata }` and, on
   the first proposal, each advertiser creative with its existing `imageKey` + `metadata` (leave
   reused images out — they are already in the set with their metadata) plus the card `heading` +
   `description` you write for this set
   (advertiser's language; heading is one sentence about this set, e.g. "Four ads for the first
   warm week, ready for your eye."; description: what to check, changes go in the correction block
   per image, approving locks the set (Approval 3/4) before media; name any advisory issue on
   their own creatives). **Every plan piece
   must have an image** — a partial set is refused and nothing is saved. The card is the hard gate
   G_CREATIVES (`AWAITING_CREATIVES`); wait for it. Reopen with `preview_images_card`
   (`campaignId`). Card corrections → mode B.
6. After G_CREATIVES (status `PLANNING_MEDIA`) tili saves the approved images to the workspace
   media library itself — nothing to upload or bind. Go on to `campaign-media-strategy`.

## Rules

- DO NOT generate before G_ASSETS or before product-image source is decided
- DO NOT render, regenerate, or replace an advertiser creative (`source: "advertiser"`) unless
  the advertiser sent a correction for it on the Creatives card
- DO NOT open the Creatives card with pieces missing, and DO NOT start media before G_CREATIVES
- DO NOT search for or add references here — they live on the approved plan
- DO NOT rewrite prompts here — a prompt change is a new assets plan: Go back card to `assets`
  (`propose_campaign_go_back`, see `campaign-workflow`), then G_ASSETS
- DO NOT render images on the root when subagents are available; root orchestrates and shows the
  set — render subagents render + upload + verify, the brand-compliance subagent reviews
- DO NOT copy creatives to any design tool — the approved image in tili storage
  (`upload_campaign_image`) is what gets published
- DO NOT show the set before the brand-compliance images review — only after `pass` or 2 revise
  rounds exhausted
- DO NOT re-render, re-review, or re-send creatives without a correction; they stay as-is
- DO NOT modify a generated creative unless the change comes from a tool: Creatives card
  corrections (`confirm_creatives_review` → `get_asset_generation_input` `correction`) or the
  brand-compliance images review revise notes. Never apply image changes the advertiser types in
  chat
- Advertiser asks in chat to change an image → do not render; before G_CREATIVES reply with the
  redirect below and reopen the card with `preview_images_card`; after G_CREATIVES it needs a Go
  back card to `creatives` (`propose_campaign_go_back`)
- ONLY generate, describe (metadata), and propose; approval registers the images

## Image changes asked in chat

Do not apply them. Say (in the advertiser's language), then call `preview_images_card`:

> Image changes go through the Creatives card so nothing is changed by mistake. Press **Request a
> correction** under that image, write what should change (attach a reference if it helps), and
> press **Regenerate**. You can correct several images at once — only those will be redone.

If the change is to the prompt, references, or copy for a piece (not a fix to the generated image),
that is a new assets plan → Go back card to `assets` (`propose_campaign_go_back`) → on approve
`campaign-assets-plan` realigns the draft → G_ASSETS.

## Product-image source (hard gate)

If not decided:

> Before I create the ad images, I need your product photo choice:
> 1. You upload / send a product photo now
> 2. Confirm a photo I already found
> 3. Use an example from your Meta product catalog
>
> After that I will create the full set of ads for you to review.

Stop until they decide. Non-product creatives: confirm no product cutout is needed.

## Global generation rules

1. Product authenticity — compose OK; do not alter product details
2. Brand constraints binding; approved logo only as-is
3. Render the approved prompt + references only
4. On-brand overlays; channel/format fit; reference alignment
5. Stable `items[].id`; visual realism (no generative artifacts)
6. Render verifies every upload (full image, right aspect ratio) before returning its key

## Modes

### A. Generate pass

Parallel render → brand-compliance images review → re-render failed → show. Ask:

> Here are all the ads. To change one, press **Request a correction** under it on the card and
> write the change there; or **approve all** to lock the set, then they are saved to your media
> library and I plan media.

### B. Correction pass (Creatives card)

The advertiser writes corrections (note + optional one-off references) on one or more images and
sends them together. The card records them on the creatives draft and posts a chat message listing the
creatives. That message is only the trigger — the change itself is read from
`get_asset_generation_input` (`correction`). An id without a pending `correction` is not rendered.

1. Read each corrected id's `correction.note` from `get_asset_generation_input` before spawning
   renders.
2. Spawn one `campaign-creative-render` subagent per corrected id, all in parallel. Brief each with
   `campaignId` + `assetId` only — `get_asset_generation_input` serves the approved prompt, plan
   references, and the correction (note, current creative as `baseImage`, correction references).
   The subagent regenerates from all of them: the result matches the approved prompt and applies
   the note (the note wins where they conflict). Corrections on an advertiser's own creative: the
   subagent edits their uploaded image from the note only.
3. Spawn `campaign-brand-compliance` (`phase: "images"`) with the corrected ids only as `images`,
   `advertiserCreativeIds`, and `corrections: [{ assetId, note }]` (the note exactly as written), so
   it checks the note is applied and everything else still matches the approved prompt. On
   `revise`, re-spawn render for those ids with the reviewer's notes — the correction stays pending,
   so render still starts from the same base (same 2-round cap).
4. `propose_creatives_review` with just those ids (fresh `metadata` for each redone image) and a
   new `heading` + `description` naming what
   was redone (e.g. "02 and 04 redone with your corrections.") → fresh Creatives card: corrected
   images replaced, every other creative unchanged. The tool refuses uncorrected ids while
   corrections are pending, and the card cannot be approved while any correction is pending.

## Examples

Input: G_ASSETS approved (4 pieces) + product photo chosen  
Output: 4 render subagents in parallel → brand-compliance images review → metadata per image →
`propose_creatives_review` → Creatives card (G_CREATIVES) → approved images saved to the media
library → media

Input: G_ASSETS approved (4 pieces); advertiser uploaded their own creative for 03  
Output: 3 render subagents (01, 02, 04) → review 01–04 (03 advisory only) → metadata for all four
→ `propose_creatives_review` with 01, 02, 04 and 03 (its existing imageKey) → Creatives card shows
all four, 03 tagged "Uploaded by you"

Input: G_ASSETS approved; advertiser uploaded their own creative for every piece  
Output: no product-image question, no render → review all (advisory) → metadata for each →
`propose_creatives_review` with every piece's existing imageKey + metadata → Creatives card

Input: 4 pieces, only 01–03 rendered, `propose_creatives_review` with 01–03  
Output: refused ("Missing: 04"), nothing saved → render 04 → review → propose all four

Input: Creatives card corrections on 02 ("too dark") and 04 (note + reference)  
Output: read both correction notes → two render subagents in parallel (02, 04) → images review of
02, 04 with `corrections` → `propose_creatives_review` with 02 and 04 only → fresh Creatives card;
01 and 03 unchanged

Input: correction on 02 ("warmer light"); review returns `revise` — light still cool, headline
moved  
Output: re-spawn render for 02 with the reviewer's note (same base) → review 02 again → propose

Input: advertiser in chat: "02 is too dark"  
Output: no render → redirect message (write it in 02's correction block on the card) →
`preview_images_card`

## Edge cases

- After revise-cap with remaining failures: still show; name what looks off
- `propose_creatives_review` refuses "needs metadata" → add `metadata` for the listed pieces
  (advertiser uploads with their existing imageKey) and propose again
- Card corrections regenerate: re-run the brand-compliance images review before re-show (same
  2-round cap per batch)
- Advertiser wants new references or a different prompt → Go back card to `assets`, then
  `campaign-assets-plan` (G_ASSETS)
- Re-approved plan after a go back: render and review only plan pieces without an image in
  `creatives`; send only those to `propose_creatives_review` — reused images stay in the set and
  still show on the Creatives card
- Go back to `creatives` approved (`PRODUCING_CREATIVES`): regenerate only the pieces the
  advertiser asked to change, review them, `propose_creatives_review` with those ids (+ metadata)
  → G_CREATIVES
- Advertiser creative looks off-brand or wrong format → still show it; name the issue in the card
  description so they can correct it on the card or keep it
- Host cannot spawn subagents → render sequentially on root with the same per-asset steps, then
  run `references/creative-review.md` inline with the same brief

## Resources

- `references/creative-review.md` — shared images checklist (same file as brand-compliance); read
  before briefing the images review, or run it inline when subagents are unavailable
- `references/asset-metadata.md` — before `propose_creatives_review`
