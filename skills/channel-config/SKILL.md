---
name: channel-config
description: >-
  Channel config phase for a META_ADS channel: draft the config (budget within the channel's
  share, audiences, placements, creative bindings) and author providers.meta from Meta ODAX
  knowledge, check it with check_channel_config, save_channel_config, wait for G_CONFIG (Approval
  4/4, approve places the channel on Meta); fix a failed publish on the config card with
  update_channel_entity. Always load when the channel is CONFIGURING, AWAITING_CONFIG, or
  PUBLISH_FAILED. Needs the channel's creatives approved (G_CREATIVES). Read
  references/media-meta.md when authoring or fixing providers.meta.
license: proprietary
---

# Channel Config

Own the channel config card and the place path for one channel (`channelId`). Common fields live
here; `providers.meta` comes from `references/media-meta.md`. Payload shape: MCP tool schema
(`save_channel_config` → `config`). Change a saved config by entity with `update_channel_entity`
(campaign, ad set, ads) instead of resending it whole.

## Workflow

1. Orient: `get_campaign_status` → the channel is `CONFIGURING` (or `AWAITING_CONFIG` /
   `PUBLISH_FAILED`); note its `plannedBudget` share and dates (null = the campaign's flight).
   Load the approved strategy (`get_campaign_strategy`) and the channel's assets, creatives and
   config (`get_channel_artifacts`) when not already in this chat (an existing config `draft` is
   the last proposal — amend it), and account context. Creatives must be `approved`
   (G_CREATIVES) — `save_channel_config` refuses otherwise (`creatives-missing` /
   `missing_creative`). **Drafted after go back** (`needsAlignment`): check audiences, bindings
   (asset ids that changed or disappeared), copy hooks, budget and `providers.meta` against the
   re-approved strategy, assets, and creatives; change only what no longer fits, then steps 3–5
   (keep geo and settings the advertiser already chose).
2. Choose the campaign type (Standard, Catalog sales, Advantage+ sales, Advantage+ catalog
   sales) with the decision table in `references/media-meta.md`, using the catalog readiness
   recorded during strategy research. Catalog ad sets need no bindings; Advantage+ needs CBO.
3. Draft common fields (audiences, budget, destinationUrl, bindings, KPIs). Keep
   `audiences[].name` stable for ad sets. Audiences come only from `list_audiences`.
   **Budget:** the daily total × the channel's flight days (or the lifetime total) must fit its
   `plannedBudget` share from the strategy (refused as `config-over-channel-budget` otherwise), and
   the average per day stays under `maxCampaignDailyBudget`. Open-ended flights: plan the daily
   total from the share and the strategy's pace, not the ceiling.
4. Read `references/media-meta.md` with campaign context + draft; merge returned
   `providers.meta`. Run `check_channel_config` (`channelId`, `config`) and fix every issue until
   it is clean; its `reach` shows each ad set's estimated audience.
5. Assemble the config → root runs the `campaign-brand-compliance` review (`phase: media`;
   Brand OS + approved strategy and assets) → apply `revise_notes` until `pass` →
   `save_channel_config` (`channelId`, `config`) → wait G_CONFIG.
6. Reopen card: `preview_channel_config_card` (`channelId`). G_CONFIG approve **places** (no
   separate publish gate); the channel becomes `LIVE`.
7. `PUBLISH_FAILED`:
   - Read why: `lastPublishError` in `get_campaign_status` or `get_channel_artifacts`, and what is
     on Meta with `get_channel_meta_state`.
   - Rate limit or auth (`next: retry_channel_publish`): the config is fine — ask the advertiser,
     then `retry_channel_publish` (`channelId`, `userConfirmed: true`) after their yes.
   - Anything else: find the smallest change Meta's error requires (fix table in media-meta),
     inside the approved strategy, assets and creatives → `check_channel_config` → compliance
     review with `publishFix` → `update_channel_entity` (or `save_channel_config`). The config
     card reopens with Meta's error and each change to what is already on Meta; Approve places it
     and patches the existing Meta objects.
   - A fix that needs other creatives, assets or a different strategy (e.g. market) →
     `propose_channel_go_back` / `propose_campaign_go_back`.

## Rules

- DO NOT invent provider configs for non-executable providers
- DO NOT propose without the `campaign-brand-compliance` review (every propose, fixes included)
- DO NOT call `retry_channel_publish` on the happy path, or for an error the config causes
- DO NOT cancel and recreate a campaign to get past a failed publish — fix it on the config card
- DO NOT change what Meta's error does not require when fixing `PUBLISH_FAILED`
- DO NOT rewrite strategy, assets, or creatives — a change there needs an approved Go back card
  (`campaign-workflow`); a different budget share is a strategy change
  (`propose_campaign_go_back`)
- DO NOT show raw provider JSON to the advertiser
- DO NOT use ad-platform jargon or codes with the advertiser — in chat, on the card, or in a fix
  `reason`: no campaign types (Advantage+, CBO/ABO), objectives, events, error codes, or ad set /
  pixel / placement talk. Say who sees the ads, where, for how much, and what went wrong in
  everyday words (`references/advertiser-language.md`)
- ONLY propose when `providers.meta` is complete (`save_channel_config` validates on propose)

## Common fields you own

Draft plain-language config fields the card needs: summary, funnel, destination URL, budget,
audiences (stable names = meta ad sets), optional placements/KPIs/constraints, and asset bindings.
Audience names describe the people in everyday words ("Visited the shop in the last 14 days",
not "RT_VC_14d").
Look/reference discovery lives on MediaAsset.metadata from creatives bind — not on the channel.
Exact field names and limits: tool schema.

## Examples

Input: `CONFIGURING`, creatives approved, channel share 4400 over 21 days  
Output: daily total ≤ 209 (e.g. 200: ABO 110 / 55 / 35) → common fields → media-meta → compliance →
`save_channel_config` → wait G_CONFIG

Input: `CONFIGURING` after go back to assets; config `draft` + `needsAlignment`; piece 04 was
dropped  
Output: remove 04 bindings, re-check audiences vs new copy, keep budget/geo → media-meta →
compliance review → `save_channel_config` → G_CONFIG

Input: range-based sales strategy with a retargeting angle; eligible catalog, pixel connected,
ViewContent and AddToCart firing  
Output: Catalog sales — retargeting audience → catalog ad set (retarget ViewContent + AddToCart,
14 days, exclude buyers 14 days); prospecting audience → broad catalog ad set; no bindings on
either → save → G_CONFIG

Input: sales strategy, one hero product, 60+ purchases a week, no per-audience budget rule  
Output: Advantage+ sales — CBO, one broad ad set with `advantage_audience: 1`, no placements,
approved creatives bound → save → G_CONFIG

Input: range-based sales strategy, healthy Purchase volume, approved static launch creatives  
Output: Advantage+ catalog sales — CBO, broad catalog ad sets only (no retargeting); or a Standard
campaign with static ad sets when the launch creatives carry the message the catalog cannot

Input: brief needs "Chișinău" and "Moldova excluding Chișinău"  
Output: ad set 1 `countries: [MD]` + `cities: [Chișinău, country MD]`; ad set 2 `countries: [MD]` +
`excluded_geo_locations.cities: [Chișinău]` → `check_channel_config` clean → compliance → save

Input: `PUBLISH_FAILED`, `lastPublishError` subcode 1487756 (conflicting locations)  
Output: `get_channel_meta_state` → tag the cities with their country / exclude the city from the
national segment → `check_channel_config` → compliance (`publishFix`) → `update_channel_entity`
(`set_ad_set` audience) → tell the advertiser "Meta didn't start the ads because two areas
overlap — I've left Chișinău out of the Moldova group; approve the fix on the card" → advertiser
approves → placed

Input: `PUBLISH_FAILED`, `lastPublishError` kind `publish-rate-limited`  
Output: "Meta asked us to slow down for a moment — nothing is wrong with your setup. Shall I try
starting the ads again?" → advertiser says yes → `retry_channel_publish` (`userConfirmed: true`)

## Edge cases

- Validation failures on save or check → fix `providers.meta` from `issues[]` codes (fix table in
  media-meta) and check again
- `meta_targeting_rejected` on check → Meta's own message on the delivery estimate; fix that ad
  set's targeting before proposing
- Catalog not ready (no eligible catalog, pixel not connected, empty product set) → Standard
  campaign; a catalog fix that writes to Meta only after the advertiser says yes in chat

## Resources

- `references/media-meta.md` — read when authoring or fixing `providers.meta`
- `references/advertiser-language.md` — before talking to the advertiser or writing card text
