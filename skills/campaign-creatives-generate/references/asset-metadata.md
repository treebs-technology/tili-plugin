# Asset metadata (pre-Canva bind)

After G_CREATIVES and before Canva bind, attach `MediaAssetMetadata` per finished creative (see
the `bind_campaign_creatives` schema). Do this before the Canva upload / binding `assetRef`.

## Rules

- DO NOT invent pixels, call Canva invent, or upload to Canva from this step
- MUST have viewed (or a faithful description of) each finished image
- MUST put real on-image copy into `text_on_image`, transcribed exactly (`"none"` if no text)
- Match each entry’s `assetId` to the assets plan id; cover every approved creative
- Descriptions must be faithful to the finished image, not aspirational

## Workflow

1. Take approved assets plan + approved creatives (`get_campaign_artifacts` `assets`, `creatives`).
2. Per piece: visual concept, image description, transcribed on-image text, tags, product, format.
3. Pass metadata per piece to `bind_campaign_creatives` with its Canva `assetRef`.
