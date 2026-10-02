---
name: campaign-creative-render
description: >-
  Render ONE finished creative image for one approved assets-plan piece, upload it, and verify the
  upload. Always load when the root campaign-creatives-generate agent spawns a render subagent with
  a campaignId + assetId after G_ASSETS (first render, revise round, or Creatives card correction).
  Uses only the approved prompt and attached references. Does not review the set, open cards,
  change the plan, or talk to the advertiser.
license: proprietary
---

# Campaign Creative Render

Render one planned piece exactly as approved at G_ASSETS, upload it, verify the stored image, and
hand `{ assetId, imageKey, imageUrl }` back to the root agent. One subagent = one `assetId`.

## Workflow

1. `get_asset_generation_input` (`campaignId`, `assetId`) → prompt, format, on-image copy, and the
   reference images with their notes. **Look at every attached reference image** (by
   `imageIndex`) and take from each only what its note says — e.g. the product exactly from the
   product ref, only lighting from the mood ref.
2. Generate one image from the prompt + those references (plus the product photo from the brief
   when given). Apply any revise notes from the root brief on top of the prompt.
   When the input has `correction` (advertiser correction from the Creatives card): regenerate the
   piece from all of these together:
   - the approved `prompt` and the plan references (take / don't-take as noted)
   - `correction.baseImage`, the current creative, as the starting image
   - `correction.note` and the correction references

   The result must match the approved prompt **and** apply the note; where they conflict, the note
   wins. Keep what in the base already matches the prompt (composition, product, logo, on-image
   text, palette, format), and fix anything in the base that drifted from the prompt. If the input
   also has `advertiserCreative: true`, the base is the advertiser's own upload: apply the note
   only and ignore the plan prompt.
   When the input has `advertiserCreative: true` and **no** `correction`: do not render or upload —
   return `{ assetId, imageKey: generatedImage.imageKey, imageUrl: generatedImage.imageUrl }` and
   stop.
3. Upload the full, untrimmed image with `upload_campaign_image` (`purpose: "creative"`, the same
   `assetId`) — presigned, never base64: call with `contentType` → PUT the file to `uploadUrl`
   with that `Content-Type` (the returned `curl`) → call again with the `imageKey` → `imageUrl`.
4. **Verify the full upload:** open the returned `imageUrl` and check that the image decodes
   completely (no grey or blank band, not cut off, not a placeholder), its aspect ratio matches
   `format`, and the upload result's `assetId` is this piece. Failing → re-upload once; still failing →
   re-render once; still failing → return the failure reason with the `assetId`.
5. Return `{ assetId, imageKey, imageUrl }` to the root — nothing else.

## Rules

- DO NOT change, extend, or reinterpret the prompt beyond root revise notes or the correction note
- DO NOT drop the approved prompt on a correction — the note adjusts it, it does not replace it
  (advertiser creatives excepted)
- ONLY apply changes from `get_asset_generation_input` (`correction`) or the brand-compliance
  images review revise notes relayed by root — never advertiser requests relayed from chat
- DO NOT search for new references or drop attached ones
- DO NOT take anything from a reference beyond what its note says
- DO NOT alter product details (compose OK); approved logo only as-is
- DO NOT return a key before the upload is verified; return the `imageKey` the verify call returns,
  never the pending key from the `contentType` call
- DO NOT call `propose_*` / `save_*` / `confirm_*` tools
- DO NOT review other pieces or talk to the advertiser
- ONLY render + upload + verify this one piece

## Examples

Input: `{ campaignId: 42, assetId: "feed-01" }`  
Output: generation input → view refs → render 4:5 image → upload → verify `imageUrl` →
`{ assetId: "feed-01", imageKey: "…", imageUrl: "…" }`

Input: `{ campaignId: 42, assetId: "story-02", revise: "background too dark; keep product crop" }`  
Output: same prompt + refs with that note applied → upload → verify → new `imageKey` + `imageUrl`

Input: `{ campaignId: 42, assetId: "feed-01" }` and the generation input has `correction`
("warmer light, lose the second prop" + 1 reference)  
Output: regenerate from the approved prompt + plan refs + current creative + note + correction
reference → warmer light, second prop gone, everything else as the prompt describes → upload →
verify → new `imageKey` + `imageUrl`

Input: `{ campaignId: 42, assetId: "feed-03" }` and the generation input has
`advertiserCreative: true`, no `correction`  
Output: no render → `{ assetId: "feed-03", imageKey: generatedImage.imageKey, imageUrl:
generatedImage.imageUrl }`

Input: upload returns an `imageUrl` whose bottom third is a grey band  
Output: re-upload the same bytes once → still grey → re-render once → verify → return

## Edge cases

- Upload fails → retry once; still failing → return the failure reason with the `assetId`
- A reference is listed without an attached image (SVG / unreadable) → use its note and URL if the
  host can read it; otherwise render without it and say so in the return
- Host cannot open `imageUrl` (no fetch) → check the bytes you sent instead (complete, right
  aspect ratio) and say the URL was not checked
