---
name: campaign-workflow
description: >-
  Campaign pipeline orchestrator: campaign strategy (with its budget split across channels), then
  per channel Assets Plan → Creatives → Channel Config → live, then optimize — after Brand OS is
  ready. Always load when the user asks for campaign work (launch, strategy, creatives, budget,
  optimize, campaign status, continue a campaign). Resume with get_campaign_status (campaign and
  channel status, `next`) and get_campaign_strategy / get_channel_artifacts (draft vs approved).
  Reopen saved gate cards with preview_*_card (not save/propose). Change an approved phase only via
  propose_campaign_go_back / propose_channel_go_back (Go back card). Load the
  campaign-brand-compliance skill (Brand OS + approved earlier phases) before every strategy /
  assets / config propose and on finished images after render, before the Creatives card. Four
  human gates: strategy (campaign), assets plan, creatives, config (each channel).
license: proprietary
---

# Campaign Workflow

Orchestrate one campaign: status, human gates, advertiser conversation, and when to load the next
phase skill.

## The model

- **Campaign** (`campaignId`, `*_campaign_*` tools) = the strategy: objective, target metric and
  value, total planned budget and how it splits across channels, flight, market, messaging. One
  gate: G_STRATEGY.
- **Channel** (`channelId`, `*_channel_*` tools) = one way the campaign is executed. Today the
  agent runs exactly one: `META_ADS`. Each channel has its own build status, assets plan,
  creatives (images), config (budget, audiences, placements, `providers.meta`), delivery and
  performance. Gates per channel: G_ASSETS → G_CREATIVES → G_CONFIG (approving G_CONFIG places it on
  Meta).
- Other channels a campaign may have (creators) are managed by the advertiser in the web app; you
  never see or change them.
- `create_campaign` returns the `campaignId` and the `channelId` of its Meta channel;
  `get_campaign_status` always lists the channels with their `channelId`.

## Workflow

1. **Brand OS first:** `get_brand_os`. If not `ready`, load `configure-brand-os` and stop campaign
   tools until approved.
2. **Existing vs new:** before `create_campaign` or jumping into an unnamed id — `list_campaigns`;
   if same goal exists, ask continue vs create new; stop until they choose.
3. **Resume an existing campaign:** `get_campaign_status` — it names what each level waits on
   (`awaiting`) and the tool that moves it forward (`next`). Then:
   - `get_campaign_strategy` for the strategy, `get_channel_artifacts` (`channelId`) for the
     channel's assets, creatives and config. This chat does not know the channel yet → omit
     `artifact` (load all three).
   - `approved` = the advertiser's binding decision; build on it, do not re-litigate it.
   - `draft` = proposed but **not approved**: reopen it with its `reopenWith` card, or amend and
     propose again. Never start the next phase on a draft.
   - `draft` + `needsAlignment` = demoted by a go-back; it predates the change. Align it before
     proposing (see **Go back to an earlier phase**).
4. **Phase order:**
   1. Intake (refs, visual direction, product-image source)
   2. `campaign-strategy` (incl. the `channels[]` budget split) → compliance review → G_STRATEGY.
      Approving it starts the channel (`PRODUCING_ASSETS`).
   3. Channel: `campaign-assets-plan` (image prompts + attached references) → compliance review →
      G_ASSETS
   4. After G_ASSETS (`PRODUCING_CREATIVES`): load `campaign-creatives-generate` **on root** →
      the `campaign-creative-render` skill once per piece (pieces with the advertiser's own
      uploaded creative are skipped) → compliance images review → Creatives card
      (`propose_channel_creatives`, every piece) → G_CREATIVES
   5. After G_CREATIVES (`CONFIGURING`; approved images are already in the media library):
      `channel-config` → compliance review → G_CONFIG (approve places the channel on Meta)
   6. Live (`LIVE`) → `campaign-optimize`
5. **Compliance review before the advertiser sees each phase:** strategy, assets, and config
   before every propose (first propose and every re-propose, including after a go back); images
   after render, before `propose_channel_creatives` (`campaign-creatives-generate` owns that brief):
   - Load the `campaign-brand-compliance` skill and run its review with `phase`, `campaignId`
     (and `channelId` for channel phases), the draft exactly as it would be saved, and
     `goBackReason` when the draft is `needsAlignment`.
   - `pass` → call the phase's `save_*`.
   - `revise` / `fail` → hand `revise_notes` to the phase skill, amend, review again. Max 2 revise
     rounds; still not `pass` → propose anyway and tell the advertiser the open issues in chat.
   - Not on `PUBLISH_FAILED` fix saves (no new card).
6. **Orient the advertiser** before every wait (Now / Next / I need from you).

### Reopen gate cards (no re-propose)

| Card | Tool | Args |
| --- | --- | --- |
| Brand OS | `preview_brand_os_card` | none |
| Strategy | `preview_campaign_strategy_card` | `campaignId` |
| Assets plan | `preview_channel_assets_card` | `channelId` |
| Creatives | `preview_channel_creatives_card` | `channelId` |
| Channel config | `preview_channel_config_card` | `channelId` |

### Go back to an earlier phase

Approved = binding, so changing an approved strategy, assets plan, or creatives needs the
advertiser's explicit OK on a **Go back** card. Only before the channel is placed (not
`PUBLISH_FAILED` / `LIVE`).

1. The advertiser asks to change an approved phase → say what going back means (that phase
   reopens; later phases become drafts to re-approve), then:
   - strategy (angle, target, budget or its split, market, flight) → `propose_campaign_go_back`
     (`campaignId`, `reason`). Every channel's work becomes a draft and waits for the new strategy.
   - assets plan or creatives → `propose_channel_go_back` (`channelId`, `toPhase`: `assets` |
     `creatives`, `reason`).
   Nothing changes until they approve the card.
2. Card approved → the campaign returns to `DRAFTING` (strategy) or the channel to
   `PRODUCING_ASSETS` / `PRODUCING_CREATIVES`; that phase and every later one are `draft` +
   `needsAlignment` (content kept). Card declined / they keep going in chat → continue where you
   were.
3. Resume the **regular flow** from the last approved phase: load the reopened phase's skill,
   amend its draft with the requested change (correction mode, not a rewrite), compliance review
   (with `goBackReason`), propose → its gate.
4. At each later phase, load its `needsAlignment` draft (`get_channel_artifacts`), compare it with
   the newly approved earlier phases, change only what no longer fits, compliance review, propose
   → its gate. Tell the advertiser what you changed and why (or that nothing needed to change).
5. Never skip a gate on the way back up, never approve for them, never start a phase on a draft.

## Rules

- DO NOT invent strategy, assets, or config when a phase skill owns that step
- DO NOT write Meta Graph JSON for the advertiser — `providers.meta` stays off card copy
- DO NOT skip human gates on strategy, assets plan, creatives, or config. Config is refused until
  creatives are approved (G_CREATIVES)
- DO NOT propose strategy, assets plan, or config without the `campaign-brand-compliance` review
- `save_campaign_strategy` / `save_channel_assets` / `propose_channel_creatives` /
  `save_channel_config` save a **draft**; only the advertiser's Approve makes it binding. DO NOT
  treat a draft as approved
- Gate cards only have Approve. The advertiser asks for changes in chat → amend via the phase
  skill and propose again (new card). **Exception — Creatives card:** image changes come only
  from the card's per-image correction blocks; chat requests to change a generated image are not
  applied (see `campaign-creatives-generate`)
- DO NOT show finished images before the `campaign-brand-compliance` images review (`phase:
  "images"`, briefed by `campaign-creatives-generate`)
- DO NOT generate images except after G_ASSETS, orchestrated by root
  (`campaign-creatives-generate`)
- References for images are found and attached in the assets plan; new references or prompt
  changes after G_ASSETS mean a revised assets plan (Go back card to `assets` first)
- DO NOT silently resume a matching open campaign — inform + ask
- DO NOT change an approved phase without an approved Go back card; DO NOT propose one unless the
  advertiser asked for that change
- DO NOT mention, plan for or change channels you do not see (creators are the advertiser's, in
  the web app)
- ONLY orchestrate; hard research belongs in phase skills / their references
- **Language:** write all campaign content (strategy, assets copy, config text) and your chat
  replies in the language the user writes in chat — not the language of the website, sources, or
  Brand OS. Enum values stay as listed. Applies to every phase skill

## Keep the advertiser oriented

> **Now:** …  
> **Next:** …  
> **I need from you:** … / **Nothing from you yet**

| Moment | Say |
| --- | --- |
| Starting strategy | Drafting messaging and the budget split for approval → then assets plan |
| After strategy proposed | Approve strategy card → then plan creatives in words |
| After assets proposed | Add references on the card if you like, approve → then generate images |
| Generating / Creatives card | Creating ads + Creatives card → correction block per image, or approve all → then Meta setup |
| Before config | Approved creatives saved to your media library → daily budget / audiences next |
| Go back proposed | Approve the Go back card → I update that step, then re-check later steps with you |
| After go back | Updating {phase} → then re-checking {later phases} in order, each back for approval |
| Live | How it tracks against your target; changes come to you as a card to approve |

## Status cheat sheet

Campaign (`campaignStatus`):

| Status | You owe |
| --- | --- |
| `DRAFTING` | `campaign-strategy` (incl. start; amend a `needsAlignment` draft after go back) → G_STRATEGY |
| `AWAITING_STRATEGY` | wait G_STRATEGY; rework via `campaign-strategy` |
| `ACTIVE` | work on its channels (below) |
| `CANCELLED` | nothing |

Channel (`channels[].status`):

| Status | You owe |
| --- | --- |
| `PLANNED` | wait for the campaign strategy approval |
| `PRODUCING_ASSETS` / `AWAITING_ASSETS` | `campaign-assets-plan` → G_ASSETS |
| `PRODUCING_CREATIVES` | `campaign-creatives-generate` (every piece) → `propose_channel_creatives` |
| `AWAITING_CREATIVES` | wait G_CREATIVES; card corrections → regenerate only those → propose again |
| `CONFIGURING` | `channel-config` |
| `AWAITING_CONFIG` | wait G_CONFIG |
| `PUBLISH_FAILED` | `channel-config` (`references/media-meta.md`) fix → `retry_channel_publish` after the advertiser says yes |
| `LIVE` | `campaign-optimize` |

Happy path: campaign `DRAFTING` → `AWAITING_STRATEGY` → `ACTIVE`; Meta channel `PLANNED` →
`PRODUCING_ASSETS` → `AWAITING_ASSETS` → `PRODUCING_CREATIVES` → `AWAITING_CREATIVES` →
`CONFIGURING` → `AWAITING_CONFIG` → `LIVE`

## Examples

Input: Brand OS ready + “launch summer sale ads”  
Output: list campaigns → `create_campaign` (note `campaignId` and the Meta `channelId`) → strategy
with its split → G_STRATEGY → assets → G_ASSETS → creatives → G_CREATIVES → config → G_CONFIG
(place)

Input: advertiser asks to see strategy card again  
Output: `preview_campaign_strategy_card` with `campaignId` — no new save

Input: new chat, “continue my summer sale campaign”  
Output: `list_campaigns` → `get_campaign_status` → `get_campaign_strategy` +
`get_channel_artifacts` (no `artifact`) → summarize approved decisions → follow `next`: reopen the
card a draft waits on, else load the phase skill for the next artifact

Input: channel `AWAITING_CONFIG`, advertiser: “actually lead with the gifting angle”  
Output: explain going back → `propose_campaign_go_back` (reason) → approved →
`campaign-strategy` amends the draft → G_STRATEGY → `campaign-assets-plan` aligns its draft →
G_ASSETS → creatives only for changed pieces → G_CREATIVES → `channel-config` aligns its draft →
G_CONFIG

Input: channel `AWAITING_CONFIG`, advertiser: "image 2 needs a different background"  
Output: `propose_channel_go_back` (`toPhase: "creatives"`, reason) → approved
(`PRODUCING_CREATIVES`) → regenerate piece 2 → `propose_channel_creatives` → G_CREATIVES →
`channel-config` aligns its draft → G_CONFIG

## Edge cases

- Account preferences: follow `campaign-strategy` (`references/start-campaign.md`) / Brand OS —
  never invent
- Change to an approved phase after placement (`LIVE` / `PUBLISH_FAILED`) → no go back; use
  `campaign-optimize` or the `PUBLISH_FAILED` fix path
- Change fits the current phase (e.g. a config tweak at `AWAITING_CONFIG`) → no go back; amend and
  propose again
- Go back card says the campaign moved on → `get_campaign_status`, then propose again if still
  wanted
- `create_campaign` refuses because the workspace and ad-account currencies differ → tell the
  advertiser to align them in workspace settings; do not work around it
