---
name: campaign-media-strategy
description: >-
  Media Plan phase: draft common MediaStrategy fields, author providers.meta from Meta ODAX
  knowledge, save_media_strategy, wait for G_MEDIA (Approval 4/4, approve places). Always load
  when PLANNING_MEDIA, AWAITING_MEDIA, or PUBLISH_FAILED. Needs approved (G_CREATIVES) and bound
  creatives. Read references/media-meta.md when META is locked or fixing Meta publish errors.
license: proprietary
---

# Campaign Media Strategy

Own the Media Plan card and place path. Common fields live here; `providers.meta` comes from
`references/media-meta.md`. Payload shape: MCP tool schema.

## Workflow

1. Orient: confirm media phase. Load approved strategy, assets, and creatives via
   `get_campaign_artifacts` when not already in this chat (an existing media `draft` is the last
   proposal — amend it), account context, locked providers. Creatives must be `approved` with an
   `assetRef` on every item; if any is missing, finish the Canva bind first
   (`campaign-creatives-generate` mode B → `bind_campaign_creatives`) — `save_media_strategy`
   refuses otherwise (`creatives-missing` / `unarchived_creative`). **Drafted after go back**
   (`needsAlignment`): check audiences, bindings (asset ids that changed or disappeared), copy
   hooks, and `providers.meta` against the re-approved strategy, assets, and creatives; change
   only what no longer fits, then steps 3–5 (keep budget, geo, and settings the advertiser already
   chose). Only proceed when locked providers are executable (see `list_accessible_providers`).
2. Choose the campaign type (Standard, Catalog sales, Advantage+ sales, Advantage+ catalog
   sales) with the decision table in `references/media-meta.md`, using the catalog readiness
   recorded during strategy research. Catalog ad sets need no bindings; Advantage+ needs CBO.
3. Draft common fields (audiences, budget, destinationUrl, bindings, KPIs). Keep
   `audiences[].name` stable for ad sets.
4. When `META` locked: read `references/media-meta.md` with campaign context + draft; merge
   returned `providers.meta`. Review; redo if incomplete.
5. Assemble `MediaStrategy` → root runs the `campaign-brand-compliance` review (`phase: media`;
   Brand OS + approved strategy and assets) → apply `revise_notes` until `pass` →
   `save_media_strategy` → wait G_MEDIA.
6. Reopen card: `preview_media_card` (`campaignId`). G_MEDIA approve **places** (no separate
   publish gate).
7. `PUBLISH_FAILED`: re-read media-meta with the error → save (no new card) → ask the advertiser
   whether to place it again → call `publish_campaign` with `userConfirmed: true` only after they
   say yes in this conversation.

## Rules

- DO NOT invent provider configs for non-executable providers
- DO NOT propose without the `campaign-brand-compliance` review (every propose, not
  `PUBLISH_FAILED` fix saves)
- DO NOT call `publish_campaign` on the happy path
- DO NOT open a new gate card while fixing `PUBLISH_FAILED`
- DO NOT rewrite strategy, assets, or creatives — a change there needs an approved Go back card
  (`campaign-workflow`)
- DO NOT show raw provider JSON to the advertiser
- ONLY propose when `providers.meta` is complete (`save_media_strategy` validates on propose)

## Common fields you own

Draft plain-language media plan fields the card needs: summary, funnel, destination URL, budget,
audiences (stable names = meta ad sets), optional placements/KPIs/constraints, and asset bindings.
Look/reference discovery lives on MediaAsset.metadata from creatives bind — not on the campaign.
Exact field names and limits: tool schema.

## Examples

Input: `PLANNING_MEDIA` + META locked + creatives bound  
Output: common fields → media-meta → compliance → `save_media_strategy` → wait G_MEDIA

Input: `PLANNING_MEDIA`, creatives approved but not yet bound to Canva  
Output: do not draft media yet → Canva archive → `bind_campaign_creatives` → then media

Input: `PLANNING_MEDIA` after go back to assets; media `draft` + `needsAlignment`; piece 04 was
dropped  
Output: remove 04 bindings, re-check audiences vs new copy, keep budget/geo → media-meta →
compliance review → `save_media_strategy` → G_MEDIA

Input: range-based sales strategy with a retargeting angle; eligible catalog, pixel connected,
ViewContent and AddToCart firing  
Output: Catalog sales — retargeting audience → catalog ad set (retarget ViewContent + AddToCart,
14 days, exclude buyers 14 days); prospecting audience → broad catalog ad set; no bindings on
either → save → G_MEDIA

Input: sales strategy, one hero product, 60+ purchases a week, no per-audience budget rule  
Output: Advantage+ sales — CBO, one broad ad set with `advantage_audience: 1`, no placements,
approved creatives bound → save → G_MEDIA

Input: range-based sales strategy, healthy Purchase volume, approved static launch creatives  
Output: Advantage+ catalog sales — CBO, broad catalog ad sets only (no retargeting); or a Standard
campaign with static ad sets when the launch creatives carry the message the catalog cannot

Input: `PUBLISH_FAILED` with targeting error  
Output: media-meta fix → save (no card) → ask to place again → advertiser says yes →
`publish_campaign` (`userConfirmed: true`)

## Edge cases

- Locked providers not executable → refuse; ask to lock an executable provider (typically META)
- Validation failures on save → fix `providers.meta` from `issues[]` codes (fix table in
  media-meta) and propose again
- Catalog not ready (no eligible catalog, pixel not connected, empty product set) → Standard
  campaign; a catalog fix that writes to Meta only after the advertiser says yes in chat

## Resources

- `references/media-meta.md` — read when authoring or fixing `providers.meta`
