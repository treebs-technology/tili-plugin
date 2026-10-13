# Media checklist (`phase: media`, Meta)

Score a `MediaStrategy` draft (common fields + `providers.meta`) before `save_channel_config`.
Meta doctrine lives in
[`channel-config/references/media-meta.md`](../../channel-config/references/media-meta.md)
— check against it, do not restate it.

## Inputs to load

- `get_brand_os`: claims rules, placements or channels Brand OS rules out, never-dos
- `get_campaign_strategy` and `get_channel_artifacts` `artifact: "assets"` — both must be
  `approved`
- `get_campaign_status`: the channel's `plannedBudget` share and dates
- The `draft` and `goBackReason` from the brief
- `get_pixel_stats` when the draft retargets or optimizes for a pixel event
- `list_custom_conversions` when an ad set sets `customConversionId`
- `publishFix` / `change` from the brief, when present

## Checklist

1. **`Alignment: media ODAX`** — pass when objective + destination + `optimization_goal` form a
   row in the media-meta v1 ODAX allowlist and fit the strategy `objective`.
2. **`Alignment: media ids`** — pass when detailed targeting, place (with its `country`), catalog,
   product-set and custom conversion ids came from lookup tools (not typed from memory).
3. **`Alignment: media bidding & events`** — pass when billing is `IMPRESSIONS`; a bid cap, cost
   cap or minimum ROAS is set only for a cost or return limit in the strategy, constraints or
   advertiser's words; and the conversion event (and any custom conversion) fits the strategy
   `objective` and fires in `get_pixel_stats`.
4. **`Alignment: media budget`** — pass when `budget.mode`, `budget.period` and the total match the
   Meta campaign / ad-set budgets, and the spend over the channel's flight (daily total × days, or
   the lifetime total) fits its `plannedBudget` share (when the flight ends).
5. **`Alignment: media ad sets`** — pass when there is exactly one ad set per `audiences[].name`.
6. **`Alignment: assets bindings`** — pass when every static ad set has at least one binding,
   catalog ad sets have none, and every binding points to an asset id in the approved assets plan.
7. **`Alignment: media Advantage+`** (only with `advantagePlus`) — pass when the campaign uses
   CBO, every ad set has `advantage_audience: 1`, and there are no `placements`.
8. **`Alignment: media DSA`** — pass when an EU/EEA target sets `campaign.dsa.beneficiary`.
9. **`Alignment: media retargeting`** — pass when retargeting uses only events `get_pixel_stats`
   shows firing.
10. **`Alignment: strategy audiences`** — pass when audiences, ages and detailed targeting match the
    strategy `audience` and angles.
11. **`Alignment: strategy market`** — pass when every ad set's `countries` are inside the strategy
    `market`, and every region, city, zip and pin sits in one of them.
12. **`Alignment: strategy funnel stage`** — pass when `funnel_stage` is the strategy's.
13. **`Alignment: media schedule`** — pass when ad-set schedules (and a lifetime budget's run) fall
    within the channel's dates from `get_campaign_status`.
14. **`Alignment: media geo segments`** — pass when ad sets do not overlap: a broad segment excludes
    the places that have their own segment, and every excluded place sits inside what is targeted.
15. **`Alignment: change scope`** (with `publishFix` or `change` only) — pass when every difference
    from `previousConfig` is required by `lastPublishError` or asked for in `request`; approved
    segments, geo, ages, budget and bindings stay otherwise. A fix that needs other creatives or a
    different strategy → `fail` naming the go-back.
16. **`Brand OS: claims and placements`** — pass when summary, catalog template copy, and
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
- "Chișinău" and "Moldova" ad sets both include Chișinău → `Alignment: media geo segments` fails →
  `revise`: "`providers.meta.adSets[1]`: add Chișinău to `excluded_geo_locations`".
- Fix for 1487756 also raises the budget from 20 to 30 → `Alignment: change scope` fails →
  `revise`: "keep `budget.dailyTotal` 20; the error is about locations".
