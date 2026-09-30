# Creative review checklist (`phase: images`)

Score the **final pixels** of finished creatives before `propose_creatives_review`. One checklist,
shared by `campaign-brand-compliance` (runs it) and `campaign-creatives-generate` (briefs it, or
runs it inline when the host cannot spawn subagents). Score only — never regenerate, upload, or
talk to the advertiser.

## Inputs to load

- The brief: `images: [{ assetId, imageKey, imageUrl }]`, `advertiserCreativeIds`, and
  `corrections: [{ assetId, note }]` on a correction pass
- `get_asset_generation_input` per id: approved prompt, `visual_direction`, format, on-image copy,
  reference images with their notes; `generatedImage` for advertiser creatives; `correction`
  (`note`, references, `baseImage`) for corrected ids
- `get_brand_os`: `section: "logos"` (marks + `use` notes), palette, type, never-dos
- `get_campaign_artifacts` `artifact: "strategy"` (approved) for the `strategy` area
- Open every image from `imageUrl` (or the inline attachment) and look at it at full size

## Checklist

Score each image on every area that applies. Record each result as `checks[].id` =
`<assetId>:<area>`.

1. **`logo`** — when the piece shows a logo: it is the Brand OS mark in the variant its `use`
   note names; not redrawn, warped, recoloured, cropped, re-spelled, or given effects; legible; no
   invented or extra logos. When the plan has no logo, none appears.
2. **`prompt_alignment`** — the image matches the approved prompt and `visual_direction`: subject
   and action, setting, shot and framing, lighting, Brand OS palette roles, style, format and safe
   zones. Quoted on-image text appears exactly once and is spelled correctly; otherwise the
   requested negative space is kept. Each reference's "take" is applied and nothing from its
   "don't take" leaks in.
3. **`product_authenticity`** — shape, proportions, label text, colour, and packaging match the
   product reference or source. Crop and angle changes are OK; no invented variants or label text.
   Skip when the piece has no product.
4. **`visual_realism`** — natural and believable for its style:
   - **Anatomy**: right number of arms, legs, hands, and fingers (five per hand, none fused or
     extra); symmetrical faces with natural eyes, teeth, and ears; limbs connect to bodies; poses
     physically possible; skin texture natural, not waxy or plastic.
   - **Objects and physics**: correct geometry, nothing melted, bent, or merged; shadows and
     reflections match the light source; plausible scale; objects rest or are held believably (no
     floating products, no hands passing through objects).
   - **No duplicates**: nothing unintentionally repeated (two products, twin faces, repeated
     props).
   - **Text**: no garbled or pseudo text anywhere (signage, packaging, background).
5. **`composition_complete`** — the composition is finished and clean:
   - focal subject fully in frame unless the prompt calls for a crop; nothing important cut at
     the edges
   - no half-rendered or missing areas, blank patches, or abrupt cut-offs
   - no generative artifacts: seams, smears, blotches, noisy or blurred patches, warped edges,
     halos or fringing around the product, stray lines, watermark, or signature
   - background coherent edge to edge
6. **`brand`** — Brand OS palette, type tone of any overlay, and never-dos respected.
7. **`strategy`** — the piece fits the approved strategy angle and key message it was planned for.
8. **`layout_composition`** — placement matches the visual direction and keeps the format's safe
   zones free for platform UI.
9. **`correction`** (only ids in `corrections`) — compare the new image with the approved prompt,
   `correction.baseImage`, and the note:
   - every change the note asks for is visible ("warmer light" → visibly warmer; "remove the
     second prop" → it is gone)
   - correction references are used only as the note says
   - everything the note does not mention matches the approved prompt; what already matched it in
     the base (composition, product, logo, on-image text, palette, format) is kept
   - where the note conflicts with the prompt, the note wins — only for what it names
   - no new issues were introduced

### Corrected ids

A correction never waives the other areas — corrected ids get the full checklist, and
`prompt_alignment` is scored against the approved prompt as adjusted by the note. For a corrected
advertiser creative the base is their upload: skip `prompt_alignment`, and score `correction`,
`logo`, `product_authenticity`, `brand`, and `visual_realism` as advisory.

### Advertiser's own creatives

Ids in `advertiserCreativeIds` are scored like the rest but are **advisory only**: never a failed
id, never a regenerate. Return their issues as advisory notes per id; root names them in the
Creatives card description. They never block `pass`.

### Sanity guard

An image that will not open from `imageUrl`, or looks truncated (grey or blank band, cut off),
fails that id with "re-render and re-upload". Render owns the primary upload check.

## Verdict

- `pass` — every generated image passes every applicable area.
- `revise` — one or more generated images fail; `revise_notes` gives per-asset regenerate
  instructions naming each failed id, what is wrong, and what to keep ("02: regenerate the hand —
  five fingers, natural grip; keep can, framing, and light").
- `fail` — missing inputs (no images, Brand OS logos missing when the plan shows a logo). Name the
  gap.

Prefer `revise` with actionable notes over a vague `fail`.

## Examples

- 01 headline reads "Summmer" → `01:prompt_alignment` fails → "01: fix headline spelling to
  'Summer', exactly once; keep layout".
- 03 can label is re-lettered → `03:product_authenticity` fails → "03: keep the can label exactly
  as ref 1; no invented text".
- 04 has a blurred smear bottom-left and the product halo → `04:composition_complete` fails →
  "04: clean background edge to edge, no halo around the can; keep composition".
- Correction on 02 "warmer light": light still cool, headline moved → `02:correction` fails →
  "02: light still cool; the headline moved — restore it to the top third".
