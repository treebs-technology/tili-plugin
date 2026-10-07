# Media Meta (providers.meta)

Author `MediaStrategy.providers.meta` (tili ODAX v1) from approved CampaignStrategy + common media
draft. Also the fixer for Meta `PUBLISH_FAILED`. Payload shape: MCP tool schema for
`save_media_strategy`. Root owns save / publish / cards.

## Rules

- DO NOT invent interest or geo ids — use `search_ad_interests` / `search_geo_locations`
- DO NOT invent catalog or product set ids — catalogs come from `get_account_context`, product
  sets from the advertiser
- DO NOT use pre-ODAX objectives (`LINK_CLICKS`, `CONVERSIONS`, …) — ODAX `OUTCOME_*` only
- DO NOT set bid caps / COST_CAP / MIN_ROAS — v1 is `LOWEST_COST_WITHOUT_CAP`
- DO NOT set `billing_event` other than `IMPRESSIONS`
- DO NOT author ads/creatives inside `providers.meta` (catalog `template` copy is the exception)
- DO NOT draft common card fields — reuse root `audiences[].name` as `adSets[].name`
- DO NOT enable dynamic creative, Messenger/WhatsApp, app promo, VALUE paths, special ad categories
- Budget mode and daily totals must match root common `budget`

## Strategy → ODAX map

| Strategy signal | Prefer |
| --- | --- |
| Awareness / reach | `OUTCOME_AWARENESS` + REACH \| IMPRESSIONS \| AD_RECALL_LIFT |
| Traffic / visits | `OUTCOME_TRAFFIC` + LINK_CLICKS \| LANDING_PAGE_VIEWS |
| Engagement | `OUTCOME_ENGAGEMENT` + ON_POST + POST_ENGAGEMENT |
| Leads | `OUTCOME_LEADS` + ON_AD form or website + LEAD |
| Sales / purchase | `OUTCOME_SALES` + website + PURCHASE |

## v1 ODAX allowlist

| Objective | Destination | optimization_goal | Promoted |
| --- | --- | --- | --- |
| OUTCOME_AWARENESS | omit | REACH \| IMPRESSIONS \| AD_RECALL_LIFT | page |
| OUTCOME_TRAFFIC | omit / WEBSITE | LINK_CLICKS \| LANDING_PAGE_VIEWS | none |
| OUTCOME_ENGAGEMENT | ON_POST | POST_ENGAGEMENT | page |
| OUTCOME_LEADS | ON_AD | LEAD_GENERATION \| QUALITY_LEAD | page |
| OUTCOME_LEADS | omit / WEBSITE | OFFSITE_CONVERSIONS | pixel + LEAD |
| OUTCOME_SALES | omit / WEBSITE | OFFSITE_CONVERSIONS | pixel + PURCHASE |

Catalog sales and Advantage+ sales are both the OUTCOME_SALES row plus the settings below.

## Campaign type decision

Decide from the approved strategy, assets and account facts. Standard is the default.

| Type | Choose when (all true) | Config |
| --- | --- | --- |
| Catalog sales | Goal is purchases; offer is a product range, not one hero product; catalog is in `get_account_context` catalogs; the advertiser names a product set that fits the offer; `get_pixel_stats` shows ViewContent / AddToCart / Purchase firing | `campaign.catalog` + `adSets[].catalog` on every ad set |
| Advantage+ sales | Goal is purchases; `get_pixel_stats` shows healthy Purchase volume; no constraint needs per-ad-set budgets (e.g. "retargeting below 25% of spend"); no strict placement or demographic limits; no catalog retargeting | `campaign.advantagePlus: true` + CBO + `advantage_audience: 1` + no `placements` |
| Advantage+ catalog sales | Both rows above hold and every catalog ad set is broad | both configs; catalog audiences `broad` only |
| Standard | Anything else | neither |

- A strategy angle whose role is retargeting → catalog retargeting ad set; prospecting angles →
  broad catalog ad sets, or a Standard campaign with static ads when the approved creatives carry
  the message.
- `list_campaigns`: an Advantage+ sales campaign already running on the same catalog overlaps —
  say so to the advertiser before adding another.

## Catalog

- `campaign.catalog { id, name }` from `get_account_context` catalogs. The campaign stays
  OUTCOME_SALES; every ad set optimizes OFFSITE_CONVERSIONS + PURCHASE.
- `adSets[].catalog.productSet { id, name }` from the advertiser (Commerce Manager). Ask which
  set fits the offer and what it contains so the copy matches real names and prices. No fitting
  set → ask the advertiser to create one in Commerce Manager. Publish checks the set exists in
  the catalog and has products.
- Audience: `broad` (Meta picks people and products) or `retarget` with `events`
  (`ViewContent` / `AddToCart`), `lookbackDays` 1–180, optional `excludePurchasedDays`. Retarget only
  events `get_pixel_stats` shows firing.
- Template: `message`, `headline`, optional `description`, `cta`. Tokens Meta fills:
  `{{product.name}}`, `{{product.brand}}`, `{{product.price}}`, `{{product.current_price}}`,
  `{{product.description}}`. Brand OS voice applies to the fixed words around the tokens.
- No `assetBindings` for catalog ad sets — product images come from the catalog. Static ad sets in
  a non-catalog campaign still need at least one binding.

## Advantage+ sales

- Three settings together: campaign budget (CBO), `advantage_audience: 1` on every ad set, no
  `placements`. Age, gender and interests become suggestions Meta may go beyond.
- Keep geo, DSA and the conversion event as usual.
- Publish reads Meta's `advantage_state_info`; a note in the publish result means Meta did not
  classify it as Advantage+ sales — tell the advertiser it runs as a standard sales campaign.

## After a go back

- Re-approved strategy no longer about purchases → remove `advantagePlus` and `catalog`.
- Assets changed → re-check whether static or catalog ad sets still fit, and bindings.

## Workflow

1. Read strategy + root draft + account ceiling / pixel / page.
2. Choose the campaign type (table above); choose ODAX row; one ad set per audience name;
   resolve interests/geo. Optionally estimate reach.
3. Custom audiences only from `list_custom_audiences`. Conversion goals → `get_pixel_stats`.
4. EU/EEA → `campaign.dsa.beneficiary`.
5. Hand complete `providers.meta` back to root for assemble / save.

## Fix mode (`lastPublishError` or save `issues[]`)

Map error → path under `providers.meta`; patch targeting/goal/budget/DSA/destination; re-resolve
stale ids; return updated Meta config for root to save (no new gate card). Root republishes with
`publish_campaign` only after the advertiser says yes to placing it again, passing
`userConfirmed: true`.

| Code | Resolve with |
| --- | --- |
| `catalog_objective` | OUTCOME_SALES + OFFSITE_CONVERSIONS + PURCHASE on every ad set |
| `catalog_adset_required` / `catalog_missing` | catalog on every ad set, or remove `campaign.catalog` |
| `catalog_template_token` | only the tokens listed under Catalog |
| `catalog_bindings_forbidden` | drop bindings to catalog ad sets |
| `audience_binding_required` | bind at least one asset to each static ad set |
| `catalog_unknown` | `get_account_context` → pick a listed catalog |
| `product_set_unknown` / `product_set_empty` | ask the advertiser for a set in that catalog with products |
| `catalog_pixel_not_connected` | ask the advertiser to connect the pixel to the catalog in Commerce Manager |
| `catalog_event_weak` | `get_pixel_stats` → retarget firing events, or switch to `broad` |
| `advantage_plus_objective` | OUTCOME_SALES or drop `advantagePlus` |
| `advantage_plus_budget` | CBO or drop `advantagePlus` |
| `advantage_plus_placements` | remove `placements` or drop `advantagePlus` |
| `advantage_plus_audience` | `advantage_audience: 1` or drop `advantagePlus` |
| `advantage_plus_retarget` | catalog audience `broad` or drop `advantagePlus` |
