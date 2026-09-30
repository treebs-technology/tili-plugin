---
name: campaign-assets-plan
description: >-
  Produce the CampaignAssets plan for G_ASSETS: copy, nested visual_direction, one
  image_generation_prompt per piece, and the reference images attached to it. Always load when
  drafting or revising the assets plan, after G_STRATEGY, or when status is PRODUCING_ASSETS /
  AWAITING_ASSETS. References are found and attached here, not at image generation. Always read
  references/image-prompt.md before writing prompts; read references/creative-references.md when
  gathering refs (soft-skip if the advertiser already provided refs + clear direction). Does not
  generate finished images.
license: proprietary
---

# Campaign Assets Plan

Produce one complete `CampaignAssets` **plan** for `save_campaign_assets` (G_ASSETS). Each piece
carries its final image prompt plus the reference images the generator must follow — the image
phase renders exactly this. The plan holds no images: finished creatives and their Canva refs
live in the separate `creatives` artifact (G_CREATIVES). Payload shape and limits: MCP tool
schema.

## Workflow

1. Load brief: approved strategy (`get_campaign_artifacts` `artifact: "strategy"` when it is not
   already in this chat; must be `approved`), Brand OS, advertiser visual-direction / product
   notes. An existing assets `draft` there is the last proposal — amend it rather than restart.
   **Drafted after go back** (`needsAlignment`): compare every piece with the approved strategy
   (and the go-back reason when assets was the reopened phase); change only copy, prompts, or
   references that no longer fit, keep ids. Leave untouched pieces exactly as they are: the server
   keeps their images for reuse and drops images of pieces whose visual direction or prompt
   changed. Then steps 5–6.
2. **Find references** (`references/creative-references.md`, unless soft-skipped). Attach them now:
   - Brand OS section images (mood board, product shots): reuse their `imageKey` as-is.
   - Anything else (advertiser files, web, Meta posts, past tili media, Canva exports):
     `upload_campaign_image` with `purpose: "reference"` (public https `url`, or a file via the
     presigned `contentType` → PUT → `imageKey` flow) → `imageKey`.
   - Put each key in that piece's `visual_direction.references[]` with a take / don’t-take `note`.
3. **Write prompts** with `references/image-prompt.md` — one prompt per piece, the executable form
   of its `visual_direction`, naming each attached reference by index with its role ("Image 1 =
   product; Image 2 = lighting only"), and passing that doc's self-check.
4. Write every planned piece in one pass (copy + nested VD + prompt + references). Keep `id` stable
   for later generate / regenerate. Then write the card `heading` + `description` (required;
   specific to the campaign, not a generic line, in the advertiser's language):
   - `heading` — one sentence naming what this plan is, e.g. "Four creatives, one per angle, built
     for the first warm week." Not a slogan or an ad headline.
   - `description` — one or two plain sentences: what is final in this plan (copy, prompts,
     references), what is still words (no images yet), and that approving starts image generation.
5. Return the plan to root for the `campaign-brand-compliance` review (Brand OS + approved
   strategy). On `revise` / `fail` → apply its `revise_notes` to the named pieces (keep ids) and
   return again.
6. Root calls `save_campaign_assets` → wait G_ASSETS (Approval 2/4). Reopen with
   `preview_assets_card` (`campaignId`). After Approve the campaign is `PRODUCING_CREATIVES`.

## On the assets card

- Each piece shows an **Image prompt** section: the prompt is read-only for the advertiser; they can
  add or remove reference images. Their changes are saved on Approve and used at generation.
- Advertiser wants a different prompt → they say so in chat → you rewrite that prompt and propose
  again (keep ids and references).
- Each piece also has **Your own creative**: the advertiser may upload a finished image instead of
  generating one. On Approve it goes into the creatives set for that piece (`source:
  "advertiser"`) and the image phase skips it. Still write full copy + prompt + references for
  every piece.
- Advertiser sends a finished creative in chat at G_ASSETS → point them to **Your own creative** on
  that piece; do not upload it or attach it as a reference yourself.

## Rules

- DO NOT generate finished images — that is `campaign-creatives-generate` after G_ASSETS
- DO NOT leave reference finding for the image phase — every reference the generator needs is
  attached to its piece here
- DO NOT pass raw URLs as references — only `imageKey`s from `upload_campaign_image` or Brand OS
- DO NOT call Canva invent / upload to Canva / speak to the advertiser (unless root brief allows)
- DO NOT invent brand claims ruled out by Brand OS
- DO NOT return partial items — every planned piece complete on first return
- DO NOT change an approved assets plan (e.g. at `PRODUCING_CREATIVES` or `PLANNING_MEDIA`) —
  that needs an approved Go back card first (`campaign-workflow`)
- ONLY produce `CampaignAssets` plan fields (see tool schema)

## Examples

Input: strategy approved + advertiser attached 3 refs with take notes  
Output: soft-skip research; upload the 3 refs → attach per piece with notes → image-prompt → save

Input: strategy approved + no refs  
Output: creative-references → upload / reuse Brand OS keys → image-prompt → save

Input: advertiser at G_ASSETS: "make piece 2 warmer, evening light"  
Output: rewrite piece 2 prompt only (same id, same refs) → save again (new card)

Input: `PRODUCING_ASSETS` after go back to strategy (new gifting angle); assets `draft` +
`needsAlignment`, creatives already generated  
Output: pieces 1 and 3 still fit (keep copy, prompt, refs unchanged); rewrite 2 and 4 for
gifting → compliance review → save → G_ASSETS → images for 1 and 3 are reused, 2 and 4 are
rendered again

## Edge cases

- Catalog ready (recorded in strategy research) and the strategy sells a product range → plan
  fewer static pieces: catalog ads build per-product visuals from the feed. Keep static pieces for
  messages the catalog cannot carry (launch story, brand proof, prospecting hooks).
- Soft-skip still requires brand constraints from Brand OS in prompts
- A reference fails to upload → pick another or write the prompt without it; never pass the URL
- Advertiser asks for changes in chat at G_ASSETS: amend contradicted items only; keep stable ids

## Resources

- `references/image-prompt.md` — always, before writing any `image_generation_prompt` (structure,
  reference roles, text rule, self-check)
- `references/creative-references.md` — when gathering visual refs and writing their role notes
