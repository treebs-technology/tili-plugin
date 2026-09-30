# Media checklist (`phase: media`, Meta)

Score a `MediaStrategy` draft (common fields + `providers.meta`) before `save_media_strategy`.
Meta doctrine lives in
[`campaign-media-strategy/references/media-meta.md`](../../campaign-media-strategy/references/media-meta.md)
— check against it, do not restate it.

## Inputs to load

- `get_brand_os`: claims rules, placements or channels Brand OS rules out, never-dos
- `get_campaign_artifacts` for `strategy` and `assets` — both must be `approved`
- The `draft` and `goBackReason` from the brief
- `get_pixel_stats` when the draft retargets or optimizes for a pixel event

## Checklist

1. **`Alignment: media ODAX`** — pass when objective + destination + `optimization_goal` form a
   row in the media-meta v1 ODAX allowlist and fit the strategy `objective`.
2. **`Alignment: media ids`** — pass when interest, geo, catalog, and product-set ids came from
   lookup tools (not typed from memory).
3. **`Alignment: media bidding`** — pass when bidding is `LOWEST_COST_WITHOUT_CAP` and billing is
   `IMPRESSIONS`.
4. **`Alignment: media budget`** — pass when `budget.mode` and `budget.dailyTotal` match the Meta
   campaign / ad-set budgets.
5. **`Alignment: media ad sets`** — pass when there is exactly one ad set per `audiences[].name`.
6. **`Alignment: assets bindings`** — pass when every static ad set has at least one binding,
   catalog ad sets have none, and every binding points to an asset id in the approved assets plan.
7. **`Alignment: media Advantage+`** (only with `advantagePlus`) — pass when the campaign uses
   CBO, every ad set has `advantage_audience: 1`, and there are no `placements`.
8. **`Alignment: media DSA`** — pass when an EU/EEA target sets `campaign.dsa.beneficiary`.
9. **`Alignment: media retargeting`** — pass when retargeting uses only events `get_pixel_stats`
   shows firing.
10. **`Alignment: strategy audiences`** — pass when audiences match the strategy `audience`.
11. **`Brand OS: claims and placements`** — pass when summary, catalog template copy, and
    placements make no claim or use no placement Brand OS rules out.

## Verdict

- `pass` — every applicable item passes.
- `revise` — fixable by editing named fields. Note format: "`providers.meta.adSets[1]`: remove
  `placements` (Advantage+)".
- `fail` — strategy or assets not `approved`, or a required input missing.

## Examples

- Binding to `05`, not in the approved plan → `Alignment: assets bindings` fails → `revise`:
  "remove or rebind `05`".
- Germany targeted, no DSA beneficiary → `Alignment: media DSA` fails → `revise`: "set
  `campaign.dsa.beneficiary` to the advertiser's legal name".
