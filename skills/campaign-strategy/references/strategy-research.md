# Strategy research

Understand the campaign direction before drafting. First confirm the brief fits the company
(alignment), then research enough to ground every strategy field in evidence. Depth follows the
brief: a small promo needs less than a new-market launch.

## 1. Alignment check (always first)

Answer each before opening a campaign or researching further:

- **Offer** — is the product or service in the brief real and offered? (Brand OS products /
  services / positioning; the advertiser's own URLs)
- **Audience** — does Brand OS allow this audience and not refuse it?
- **Claims** — are the claims the brief implies supported, not ruled out by BINDING Brand OS?
- **Tracking** — can the account run and measure the implied objective? (account context, pixel
  events that actually fire)
- **Open work** — does an open campaign already cover this goal?

### Stop conditions

Stop and ask the advertiser, in plain language, when:

- Brand OS is not ready → load `configure-brand-os`
- the product or service is not found
- the audience or a claim is ruled out by Brand OS
- the objective is not trackable on this account
- an open campaign matches the goal (continue vs create new)
- a required fact is missing (what is sold, to whom, where, what a result looks like, daily
  spend in account currency)

Never fill a gap with industry defaults.

## 2. Research questions

Pick the questions the brief needs. Accessible tools that usually answer them are noted; use any
other accessible tool that helps you understand the direction.

- **Company, offer, positioning** — what exactly is sold, for whom, at what price, how the brand
  speaks, what it may claim. `get_brand_os` (products/services, audience, voice, claims,
  preferences); `fetch_brand_source` for advertiser-supplied URLs only.
- **Account and tracking reality** — currency, page, pixel, catalogs, providers, and which
  conversion events really fire. `get_account_context`, `get_pixel_stats`,
  `list_accessible_providers`. `get_account_context` also returns `maxCampaignDailyBudget` — sanity
  check `plannedBudget` against it (averaged over the flight) before drafting.
- **Catalog readiness** (sales goals on a product range) — does a catalog exist, which product set
  fits the offer, do ViewContent / AddToCart / Purchase fire? `get_account_context` (catalogs),
  `get_catalog_health` (products, must-fix issues, pixel link and match rates, product sets),
  `get_pixel_stats`. Record the answer in the conversation; the media phase uses it to choose the
  campaign type.
- **Customers, products, sales** — best sellers, pricing, segments, repeat behaviour, reviews.
  Ask the advertiser; workspace data sources are not readable over MCP.
- **Audience signals** — who the advertiser already targets or owns.
  `list_audiences`.
- **What the brand already says** — recurring themes, proof points, what got engagement.
  `list_facebook_posts`, `list_instagram_posts`, `list_ad_creatives`, `fetch_meta_media`,
  `list_media_assets_for_references`.
- **Campaign history** — what ran before, what performed, which objections came up.
  `list_campaigns` (each with its performance line), then `get_campaign_performance` on relevant
  ones — objective status, ROAS / CPA, what the suggestions said — and `get_channel_performance`
  when ad-set or audience results matter.

## 3. From findings to fields

- `insight`, `audience_problem`, `primary_objection` — from reviews, customer data, past results
- `reason_to_believe` — only facts found in Brand OS, data sources, or provider content
- `audience` — from customer data and existing audiences, not demographics alone
- `objective` — one the pixel can measure
- `supporting_messages` — angles that match real segments, moments, or funnel needs found above

Research contradicts the brief → back to the alignment check and ask.
