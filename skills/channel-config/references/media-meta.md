# Media Meta (providers.meta)

Author `MediaStrategy.providers.meta` (tili ODAX v1) from approved CampaignStrategy + common media
draft. Also the fixer for Meta `PUBLISH_FAILED`. Payload shape: MCP tool schema for
`save_channel_config`. Root owns save / publish / cards.

## Rules

- DO NOT invent targeting ids, place keys or locale keys — use `search_targeting`
- DO NOT invent catalog or product set ids — catalogs come from `get_account_context`, product
  sets from `get_catalog_health` or `manage_product_set`
- DO NOT use pre-ODAX objectives (`LINK_CLICKS`, `CONVERSIONS`, …) — ODAX `OUTCOME_*` only
- DO NOT set a bid cap, cost cap or minimum ROAS the advertiser has not asked for — the default is
  `LOWEST_COST_WITHOUT_CAP`
- DO NOT set `billing_event` other than `IMPRESSIONS`
- DO NOT author ads/creatives inside `providers.meta` (catalog `template` copy is the exception)
- DO NOT draft common card fields — reuse root `audiences[].name` as `adSets[].name`
- DO NOT enable dynamic creative, Messenger/WhatsApp, app promo, special ad categories
- Budget mode, period and totals must match root common `budget`

## Strategy → ODAX map

| Strategy signal | Prefer |
| --- | --- |
| Awareness / reach | `OUTCOME_AWARENESS` + REACH \| IMPRESSIONS \| AD_RECALL_LIFT |
| Traffic / visits | `OUTCOME_TRAFFIC` + LINK_CLICKS \| LANDING_PAGE_VIEWS |
| Engagement | `OUTCOME_ENGAGEMENT` + ON_POST + POST_ENGAGEMENT |
| Leads | `OUTCOME_LEADS` + website + a lead event (LEAD, COMPLETE_REGISTRATION, CONTACT, SCHEDULE, …) |
| Sales / purchase | `OUTCOME_SALES` + website + PURCHASE (or an earlier shopping event when purchases are too few) |
| Sales with value goal | `OUTCOME_SALES` + VALUE + PURCHASE, optionally a minimum ROAS |

## v1 ODAX allowlist

| Objective | Destination | optimization_goal | Promoted |
| --- | --- | --- | --- |
| OUTCOME_AWARENESS | omit | REACH \| IMPRESSIONS \| AD_RECALL_LIFT | page |
| OUTCOME_TRAFFIC | omit / WEBSITE | LINK_CLICKS \| LANDING_PAGE_VIEWS | none |
| OUTCOME_ENGAGEMENT | ON_POST | POST_ENGAGEMENT | page |
| OUTCOME_LEADS | omit / WEBSITE | OFFSITE_CONVERSIONS | pixel + lead event |
| OUTCOME_SALES | omit / WEBSITE | OFFSITE_CONVERSIONS \| VALUE | pixel + shopping event (VALUE: PURCHASE) |

Lead events: LEAD, COMPLETE_REGISTRATION, CONTACT, SUBMIT_APPLICATION, SCHEDULE, FIND_LOCATION,
START_TRIAL, SUBSCRIBE. Shopping events: PURCHASE, ADD_TO_CART, INITIATE_CHECKOUT,
ADD_PAYMENT_INFO, ADD_TO_WISHLIST, CONTENT_VIEW, SEARCH, CUSTOMIZE_PRODUCT, DONATE, START_TRIAL,
SUBSCRIBE. Optimize only for events `get_pixel_stats` shows firing. A custom conversion from
`list_custom_conversions` (`delivery.customConversionId`) narrows the event to the advertiser's
own rule, e.g. a thank-you page.

Catalog sales and Advantage+ sales are both the OUTCOME_SALES row plus the settings below.
Instant forms (`ON_AD`) are not available on this connection: ads never carry a lead form, so
leads optimize on the website.

## Targeting

- **Places.** `geo_locations.countries` are the countries the ad set serves, inside the strategy
  `market`. Narrow a country with `regions`, `cities` (optional `radius`), `zips`, or
  `custom_locations` (a pin plus radius; `name` says what it marks).
  - Every region, city, zip and pin carries the `country` and `name` that `search_targeting`
    (type `geo`) returned.
  - tili sends Meta only the narrowest places: `MD` plus Chișinău targets Chișinău only.
- **Segments do not overlap.** When one ad set targets a city and another targets the rest of
  its country, the broad one lists the country and puts that city under
  `excluded_geo_locations`.
- **Detailed targeting.** `flexible_spec` entries all apply (AND); items inside one entry are
  alternatives (OR).
  - Keys: `interests` / `behaviors` (types `interest` / `behavior`); `life_events`, `industries`,
    `income`, `family_statuses` (type `demographic`, whose result `field` names the key);
    `work_positions`, `work_employers`, `education_schools`, `education_majors`.
  - `exclusions` takes the same keys and leaves out anyone matching one item.
- **Excluded audiences.** `excluded_custom_audiences` (e.g. recent buyers) from `list_audiences`.
- **Size.** Up to 10 ad sets, and up to 6 bound creatives per ad set; extra bindings are refused.

## Delivery and budget

- **Bidding** (`delivery.bid_strategy`). `LOWEST_COST_WITHOUT_CAP` by default.
  - `COST_CAP` or `LOWEST_COST_WITH_BID_CAP` with `bid_amount` (account currency, decimals allowed)
    when the advertiser holds a cost-per-result limit.
  - `LOWEST_COST_WITH_MIN_ROAS` with `minRoas` (e.g. 2.5) and optimization goal `VALUE`.
  - A CBO campaign has one bid strategy: every ad set uses the same one.
- **Budget period.** Daily (`dailyBudget`, common `budget.dailyTotal`) or lifetime
  (`lifetimeBudget`, common `budget.period: lifetime` + `budget.lifetimeTotal`).
  - Lifetime needs `schedule.endAt` on every ad set. The ceiling check uses its average per day.
  - Never mix daily and lifetime ad sets.
- **Hours of the day** (`schedule.dayparting`): whole hours in the ad account's time zone; needs a
  lifetime budget.
- **Attribution** (`delivery.attribution`): 1- or 7-day click, optional 1-day view; conversion
  goals only. Omit for Meta's default.
- **Frequency cap** (`delivery.frequencyCap`): reach goals only, e.g. 2 impressions every 7 days.

## Campaign type decision

Decide from the approved strategy, assets and account facts. Standard is the default.

| Type | Choose when (all true) | Config |
| --- | --- | --- |
| Catalog sales | Goal is purchases; offer is a product range, not one hero product; catalog is in `get_account_context` catalogs and `get_catalog_health` shows products, no must-fix issues and the pixel connected; a product set fits the offer; `get_pixel_stats` shows ViewContent / AddToCart / Purchase firing | `campaign.catalog` + `adSets[].catalog` on every ad set |
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
- `adSets[].catalog.productSet { id, name }` from `get_catalog_health` product sets. Ask the
  advertiser which set fits the offer; `list_catalog_products` with that set shows what it holds,
  so the copy matches real names and prices. No fitting set → propose a filter (product type,
  brand, custom label), show its matches with `list_catalog_products` (filter), and create it with
  `manage_product_set` only after the advertiser says yes. Publish checks the set exists in the
  catalog and has products.
- `get_catalog_health` must-fix issues block delivery of the affected products. Product data
  changes go through `upsert_products` (retailer ids and current values from
  `list_catalog_products`), only after the advertiser says yes to the exact change. Not `finished`
  → send the same items again later (updates repeat safely); never widen the change.
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
  classify it as Advantage+ sales — tell the advertiser in plain words that Meta runs it as a
  regular sales campaign, not its fully automatic one.

## After a go back

- Re-approved strategy no longer about purchases → remove `advantagePlus` and `catalog`.
- Assets changed → re-check whether static or catalog ad sets still fit, and bindings.

## Workflow

1. Read strategy + root draft + account ceiling / pixel / page.
2. Choose the campaign type (table above); choose ODAX row; one ad set per audience name;
   resolve interests/geo with `search_targeting`. Optionally `estimate_audience` with the ad
   set's audience, placements and optimization goal.
3. Custom audiences only from `list_audiences`. Include only `ready` ones; exclusions (e.g. recent
   buyers) may be too small to deliver and still apply. Conversion goals → `get_pixel_stats`.
4. EU/EEA → `campaign.dsa.beneficiary`.
5. Hand complete `providers.meta` back to root; root runs `check_channel_config` until clean, then
   the compliance review and save.

## Fix mode (`lastPublishError`, check or save `issues[]`)

- Read Meta's message and subcode in `lastPublishError` (`get_campaign_status` /
  `get_channel_artifacts`) and what is already on Meta in `get_channel_meta_state`.
- Map the error to a path under `providers.meta` and patch only that: targeting, goal, budget,
  bidding, DSA or destination. Re-resolve stale ids with `search_targeting`.
- Return the change as `update_channel_entity` ops (or a whole config). Root runs
  `check_channel_config`, the compliance review with `publishFix`, then the tool. The config card
  reopens with the error and every change to what is on Meta; Approve places the channel and
  patches the existing Meta objects (no cancel and recreate).
- Rate limit or auth errors are not config errors: root asks the advertiser and calls
  `retry_channel_publish` with `userConfirmed: true` after their yes.

| Code | Resolve with |
| --- | --- |
| Meta 1487756 (conflicting locations) | tag every region / city / zip / pin with its `country`; split overlapping places into separate ad sets, or exclude the narrower place from the broad one |
| `geo_country_required` / `geo_country_mismatch` | copy `country` from `search_targeting`; the place's country must be in `countries` |
| `geo_overlap` | target the region or the city inside it, not both |
| `geo_exclusion_conflict` / `geo_exclusion_outside` | exclude only places inside the targeted countries, never a targeted place itself |
| `geo_radius_out_of_range` | city radius 17–80 km (10–50 mi); pin radius 1–80 km |
| `targeting_id_invalid` / `interest_id_invalid` | `search_targeting` again and use a returned id |
| `too_many_ads` | at most 6 bindings per ad set; split the creatives across ad sets |
| `lead_form_unsupported` / `messaging_destination_unsupported` | website destination: OFFSITE_CONVERSIONS + LEAD, CTA SHOP_NOW / LEARN_MORE / CONTACT_US |
| `odax_conversion_mismatch` | an event from the objective's list above |
| `bid_amount_required` / `bid_amount_forbidden` / `min_roas_*` / `value_requires_purchase` | bidding rules above |
| `cbo_budget_one` / `abo_budget_required` / `budget_period_mixed` / `budget_period_mismatch` | one budget per campaign (CBO) or per ad set (ABO), one period throughout, matching common `budget` |
| `lifetime_end_required` / `dayparting_requires_lifetime` | end dates on every ad set; hours only with a lifetime budget |
| `custom_conversion_unknown` | an active id from `list_custom_conversions` |
| `meta_targeting_rejected` | Meta's own message on the delivery estimate: fix that ad set's targeting |
| `catalog_objective` | OUTCOME_SALES + OFFSITE_CONVERSIONS + PURCHASE on every ad set |
| `catalog_adset_required` / `catalog_missing` | catalog on every ad set, or remove `campaign.catalog` |
| `catalog_template_token` | only the tokens listed under Catalog |
| `catalog_bindings_forbidden` | drop bindings to catalog ad sets |
| `audience_binding_required` | bind at least one asset to each static ad set |
| `catalog_unknown` | `get_account_context` → pick a listed catalog |
| `product_set_unknown` / `product_set_empty` | `get_catalog_health` → a set in that catalog with products, or `manage_product_set` after the advertiser's yes |
| `catalog_pixel_not_connected` | ask the advertiser, in plain steps, to link their website tracking to their product list in Meta Commerce Manager |
| `catalog_event_weak` | `get_pixel_stats` → retarget firing events, or switch to `broad` |
| `advantage_plus_objective` | OUTCOME_SALES or drop `advantagePlus` |
| `advantage_plus_budget` | CBO or drop `advantagePlus` |
| `advantage_plus_placements` | remove `placements` or drop `advantagePlus` |
| `advantage_plus_audience` | `advantage_audience: 1` or drop `advantagePlus` |
| `advantage_plus_retarget` | catalog audience `broad` or drop `advantagePlus` |
