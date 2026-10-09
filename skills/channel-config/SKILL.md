---
name: channel-config
description: >-
  Channel config phase for a META_ADS channel: draft the config (daily budget within the
  channel's share, audiences, placements, creative bindings) and author providers.meta from Meta
  ODAX knowledge, save_channel_config, wait for G_CONFIG (Approval 4/4, approve places the channel
  on Meta). Always load when the channel is CONFIGURING, AWAITING_CONFIG, or PUBLISH_FAILED. Needs
  the channel's creatives approved (G_CREATIVES). Read references/media-meta.md when authoring or
  fixing providers.meta.
license: proprietary
---

# Channel Config

Own the channel config card and the place path for one channel (`channelId`). Common fields live
here; `providers.meta` comes from `references/media-meta.md`. Payload shape: MCP tool schema
(`save_channel_config` → `config`).

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
   `audiences[].name` stable for ad sets. **Budget:** `budget.dailyTotal` × the channel's flight
   days must fit its `plannedBudget` share from the strategy (refused as
   `config-over-channel-budget` otherwise), and stays under `maxCampaignDailyBudget`. Open-ended
   flights: plan the daily total from the share and the strategy's pace, not the ceiling.
4. Read `references/media-meta.md` with campaign context + draft; merge returned
   `providers.meta`. Review; redo if incomplete.
5. Assemble the config → root runs the `campaign-brand-compliance` review (`phase: media`;
   Brand OS + approved strategy and assets) → apply `revise_notes` until `pass` →
   `save_channel_config` (`channelId`, `config`) → wait G_CONFIG.
6. Reopen card: `preview_channel_config_card` (`channelId`). G_CONFIG approve **places** (no
   separate publish gate); the channel becomes `LIVE`.
7. `PUBLISH_FAILED`: re-read media-meta with the error → `save_channel_config` (saved without a
   new card) → ask the advertiser whether to place it again → call `retry_channel_publish`
   (`channelId`) with `userConfirmed: true` only after they say yes in this conversation.

## Rules

- DO NOT invent provider configs for non-executable providers
- DO NOT propose without the `campaign-brand-compliance` review (every propose, not
  `PUBLISH_FAILED` fix saves)
- DO NOT call `retry_channel_publish` on the happy path
- DO NOT open a new gate card while fixing `PUBLISH_FAILED`
- DO NOT rewrite strategy, assets, or creatives — a change there needs an approved Go back card
  (`campaign-workflow`); a different budget share is a strategy change
  (`propose_campaign_go_back`)
- DO NOT show raw provider JSON to the advertiser
- ONLY propose when `providers.meta` is complete (`save_channel_config` validates on propose)

## Common fields you own

Draft plain-language config fields the card needs: summary, funnel, destination URL, budget,
audiences (stable names = meta ad sets), optional placements/KPIs/constraints, and asset bindings.
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

Input: `PUBLISH_FAILED` with targeting error  
Output: media-meta fix → save (no card) → ask to place again → advertiser says yes →
`retry_channel_publish` (`userConfirmed: true`)

## Edge cases

- Validation failures on save → fix `providers.meta` from `issues[]` codes (fix table in
  media-meta) and propose again
- Catalog not ready (no eligible catalog, pixel not connected, empty product set) → Standard
  campaign; a catalog fix that writes to Meta only after the advertiser says yes in chat

## Resources

- `references/media-meta.md` — read when authoring or fixing `providers.meta`
