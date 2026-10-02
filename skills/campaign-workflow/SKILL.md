---
name: campaign-workflow
description: >-
  Campaign pipeline orchestrator: order Strategy → Assets Plan → Creatives → Media Plan → optimize
  after Brand OS is ready. Always load when the user asks for campaign work (launch, strategy,
  creatives, media, optimize, campaign status, continue a campaign). Resume with
  get_campaign_artifacts (draft vs approved). Reopen saved gate cards with preview_*_card (not
  save/propose). Change an approved phase only via propose_campaign_go_back (Go back card).
  Load the campaign-brand-compliance skill (Brand OS + approved earlier phases) before every
  strategy / assets / media propose and on finished images after render, before the Creatives
  card. Four human gates: strategy, assets plan, creatives, media.
license: proprietary
---

# Campaign Workflow

Orchestrate one campaign: status, human gates, advertiser conversation, and when to load the next
`campaign-*` phase skill.

## Workflow

1. **Brand OS first:** `get_brand_os`. If not `ready`, load `configure-brand-os` and stop campaign
   tools until approved.
2. **Existing vs new:** before `create_campaign` or jumping into an unnamed id — `list_campaigns`;
   if same goal exists, ask continue vs create new; stop until they choose.
3. **Resume an existing campaign:** `get_campaign_status`, then `get_campaign_artifacts`.
   - This chat does not know the campaign yet → omit `artifact` (load everything) before drafting
     or advising.
   - Already know the rest → pass `artifact` for the slice you need (e.g. `strategy` before the
     assets plan, `assets` + `creatives` before media).
   - `approved` = the advertiser's binding decision; build on it, do not re-litigate it.
   - `draft` = proposed but **not approved**: reopen it with its `reopenWith` card, or amend and
     propose again. Never start the next phase on a draft.
   - `draft` + `needsAlignment` = demoted by a go-back; it predates the change. Align it before
     proposing (see **Go back to an earlier phase**).
4. **Phase order:**
   1. Intake (refs, visual direction, product-image source)
   2. `campaign-strategy` → compliance review → G_STRATEGY
   3. `campaign-assets-plan` (image prompts + attached references) → compliance review → G_ASSETS
   4. After G_ASSETS (`PRODUCING_CREATIVES`): load `campaign-creatives-generate` **on root** →
      the `campaign-creative-render` skill once per piece (pieces with the advertiser's own
      uploaded creative are skipped) → compliance images review → Creatives card
      (`propose_creatives_review`, every piece) → G_CREATIVES
   5. After G_CREATIVES (`PLANNING_MEDIA`; approved images are already in the media library):
      `campaign-media-strategy` → compliance review → G_MEDIA (approve places)
   6. Live → `campaign-optimize`
5. **Compliance review before the advertiser sees each phase:** strategy, assets, and media
   before every propose (first propose and every re-propose, including after a go back); images
   after render, before `propose_creatives_review` (`campaign-creatives-generate` owns that brief):
   - Load the `campaign-brand-compliance` skill and run its review with `phase`, `campaignId`, the
     draft exactly as it would be saved, and `goBackReason` when the draft is `needsAlignment`.
   - `pass` → call the phase's `save_*`.
   - `revise` / `fail` → hand `revise_notes` to the phase skill, amend, review again. Max 2 revise
     rounds; still not `pass` → propose anyway and tell the advertiser the open issues in chat.
   - Not on `PUBLISH_FAILED` fix saves (no new card).
6. **Orient the advertiser** before every wait (Now / Next / I need from you).

### Reopen gate cards (no re-propose)

| Card | Tool | Args |
| --- | --- | --- |
| Brand OS | `preview_brand_os_card` | none |
| Strategy | `preview_strategy_card` | `campaignId` |
| Assets plan | `preview_assets_card` | `campaignId` |
| Creatives | `preview_images_card` | `campaignId` |
| Media | `preview_media_card` | `campaignId` |

### Go back to an earlier phase

Approved = binding, so changing an approved strategy, assets plan, or creatives needs the
advertiser's explicit OK on a **Go back** card. Only before placement (not `PUBLISH_FAILED` /
`PUBLISHED`).

1. The advertiser asks to change an approved phase (e.g. at media: "change the strategy angle",
   "redo image 2") → say what going back means (that phase reopens; later phases become drafts to
   re-approve) → `propose_campaign_go_back` with `campaignId`, `toPhase` (`strategy` | `assets` |
   `creatives`), and `reason` (their change, one or two sentences). Nothing changes until they
   approve the card.
2. Card approved → status returns to that phase's working status (`DRAFTING`,
   `PRODUCING_ASSETS`, or `PRODUCING_CREATIVES`); that phase and every later one are `draft` +
   `needsAlignment` (content kept). Card declined / they keep going in chat → continue where you
   were.
3. Resume the **regular flow** from the last approved phase: load the reopened phase's skill,
   amend its draft with the requested change (correction mode, not a rewrite), compliance review
   (with `goBackReason`), propose → its gate.
4. At each later phase, load its `needsAlignment` draft (`get_campaign_artifacts`), compare it with
   the newly approved earlier phases, change only what no longer fits, compliance review, propose
   → its gate. Tell the advertiser what you changed and why (or that nothing needed to change).
5. Never skip a gate on the way back up, never approve for them, never start a phase on a draft.

## Rules

- DO NOT invent strategy, assets, or media when a phase skill owns that step
- DO NOT write Meta Graph JSON for the advertiser — `providers.meta` stays off card copy
- DO NOT skip human gates on strategy, assets plan, creatives, or media. Media is refused until
  creatives are approved (G_CREATIVES)
- DO NOT propose strategy, assets plan, or media without the `campaign-brand-compliance` review
- `save_campaign_strategy` / `save_campaign_assets` / `propose_creatives_review` /
  `save_media_strategy` save a **draft**; only the advertiser's Approve makes it binding. DO NOT
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
- DO NOT change an approved phase without an approved Go back card (`propose_campaign_go_back`);
  DO NOT propose one unless the advertiser asked for that change
- ONLY orchestrate; hard research belongs in phase skills / their references
- **Language:** write all campaign content (strategy, assets copy, media plan text) and your chat
  replies in the language the user writes in chat — not the language of the website, sources, or
  Brand OS. Enum values stay as listed. Applies to every phase skill

## Keep the advertiser oriented

> **Now:** …  
> **Next:** …  
> **I need from you:** … / **Nothing from you yet**

| Moment | Say |
| --- | --- |
| Starting strategy | Drafting messaging for approval → then assets plan |
| After strategy proposed | Approve strategy card → then plan creatives in words |
| After assets proposed | Add references on the card if you like, approve → then generate images |
| Generating / Creatives card | Creating ads + Creatives card → correction block per image, or approve all → then media |
| Before media | Approved creatives saved to your media library → budget / audience next |
| Go back proposed | Approve the Go back card → I update that step, then re-check later steps with you |
| After go back | Updating {phase} → then re-checking {later phases} in order, each back for approval |

## Status cheat sheet

| Status | You owe |
| --- | --- |
| `DRAFTING` | `campaign-strategy` (incl. start; amend a `needsAlignment` draft after go back) → G_STRATEGY |
| `AWAITING_STRATEGY` | wait G_STRATEGY; rework via `campaign-strategy` |
| `PRODUCING_ASSETS` / `AWAITING_ASSETS` | `campaign-assets-plan` → G_ASSETS |
| `PRODUCING_CREATIVES` | `campaign-creatives-generate` (every piece) → `propose_creatives_review` |
| `AWAITING_CREATIVES` | wait G_CREATIVES; card corrections → regenerate only those → propose again |
| `PLANNING_MEDIA` | `campaign-media-strategy` |
| `AWAITING_MEDIA` | wait G_MEDIA |
| `PUBLISH_FAILED` | `campaign-media-strategy` (`references/media-meta.md`) fix → republish after the advertiser says yes |
| `PUBLISHED` | `campaign-optimize` |

Happy path: `DRAFTING` → `AWAITING_STRATEGY` → `PRODUCING_ASSETS` → `AWAITING_ASSETS` →
`PRODUCING_CREATIVES` → `AWAITING_CREATIVES` → `PLANNING_MEDIA` (bind, then media plan) →
`AWAITING_MEDIA` → `PUBLISHED`

## Examples

Input: Brand OS ready + “launch summer sale ads”  
Output: list campaigns → strategy → G_STRATEGY → assets → G_ASSETS → creatives → G_CREATIVES →
bind → media → G_MEDIA (place)

Input: advertiser asks to see strategy card again  
Output: `preview_strategy_card` with `campaignId` — no new save

Input: new chat, “continue my summer sale campaign”  
Output: `list_campaigns` → `get_campaign_status` → `get_campaign_artifacts` (no `artifact`) →
summarize approved decisions → if the current gate's artifact is `draft`, reopen its card; else
load the phase skill for the next artifact

Input: `AWAITING_MEDIA`, advertiser: “actually lead with the gifting angle”  
Output: explain going back → `propose_campaign_go_back` (`toPhase: "strategy"`, reason) → approved →
`campaign-strategy` amends the draft → G_STRATEGY → `campaign-assets-plan` aligns its draft →
G_ASSETS → creatives only for changed pieces → G_CREATIVES → bind → `campaign-media-strategy`
aligns its draft → G_MEDIA

Input: `AWAITING_MEDIA`, advertiser: "image 2 needs a different background"  
Output: `propose_campaign_go_back` (`toPhase: "creatives"`, reason) → approved
(`PRODUCING_CREATIVES`) → regenerate piece 2 → `propose_creatives_review` → G_CREATIVES → bind →
`campaign-media-strategy` aligns its draft → G_MEDIA

## Edge cases

- Account preferences: follow `campaign-strategy` (`references/start-campaign.md`) / Brand OS —
  never invent
- Change to an approved phase after placement (`PUBLISHED` / `PUBLISH_FAILED`) → no go back; use
  `campaign-optimize` or the `PUBLISH_FAILED` fix path
- Change fits the current phase (e.g. media tweak at `AWAITING_MEDIA`) → no go back; amend and
  propose again
- Go back card says the campaign moved on → `get_campaign_status`, then propose again if still
  wanted
