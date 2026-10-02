# Asset metadata (before propose)

Before `propose_creatives_review`, write `MediaAssetMetadata` for every image you send (see the
`propose_creatives_review` schema). On G_CREATIVES approve, tili saves each image to the workspace
media library with this metadata, so later campaigns can find it
(`list_media_assets_for_references`).

## Rules

- DO NOT invent pixels or change an image in this step
- MUST have viewed (or a faithful description of) each finished image
- MUST put real on-image copy into `text_on_image`, transcribed exactly (`"none"` if no text)
- Match each entry’s `assetId` to the assets plan id; cover every image sent, including the
  advertiser's own uploads on the first proposal
- Descriptions must be faithful to the finished image, not aspirational

## Workflow

1. Take the approved assets plan (`get_campaign_artifacts` `assets`) and the finished images.
2. Per piece: visual concept, image description, transcribed on-image text, tags, product, format.
3. Pass it as `metadata` next to the piece's `imageKey` in `propose_creatives_review`.
