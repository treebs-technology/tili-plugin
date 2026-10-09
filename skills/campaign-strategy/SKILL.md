---
name: campaign-strategy
description: >-
  Check the brief against the company, research the campaign direction, then draft the messaging
  CampaignStrategy (plus market, flight schedule, planned budget and its split across channels)
  and open the campaign when needed. Always load for campaign strategy work, create_campaign,
  G_STRATEGY, or when the campaign is DRAFTING / AWAITING_STRATEGY. No Meta placements, bid
  strategy, or Graph JSON — those belong in channel-config. Read references/strategy-research.md
  before drafting and references/start-campaign.md before create_campaign.
license: proprietary
---

# Campaign Strategy

Produce one messaging `CampaignStrategy` for `save_campaign_strategy` (G_STRATEGY). It is the core
document that sets campaign direction: every channel's assets, creatives and config, and
optimisation, follow it. Approving it starts the campaign's channels.
Field meaning and limits: MCP tool schema descriptions. Never propose before the brief is aligned
and researched.

## Workflow

1. **Alignment check (first, always).** Before opening or resuming a campaign and before any
   research, check the brief against what the company actually sells and whom it serves, what
   BINDING Brand OS allows, what the account can run and track, and open work. Checklist and stop
   conditions: `references/strategy-research.md`. Misaligned or missing a key fact → stop, explain
   the gap in plain language, ask. Nothing is created and nothing drafted until they answer.
2. If no campaign id yet (or advertiser asks to start): follow `references/start-campaign.md`
   through `create_campaign`. Existing campaign: `get_campaign_status` for its channels; if this
   chat has not seen it, `get_campaign_strategy` — a `draft` is the last unapproved
   proposal (reopen or amend it, don't restart blind). A `needsAlignment` draft means the
   advertiser went back to strategy: correction mode on that draft with the change they asked
   for (skip steps 1 and 3 unless the change contradicts the brief), then steps 4–6.
3. **Research the campaign direction.** Use the accessible tools to understand the company, its
   products and services, the audience, past communication and results, and workspace data
   (`references/strategy-research.md`). You choose which sources the brief needs, and may use any
   other accessible tool that helps. Research contradicts the brief → back to step 1.
4. Draft every field as its schema description defines, grounded in research findings. Keep
   fields consistent: proposition answers `audience_problem` through `insight`; `key_message`
   derives from the proposition; `supporting_messages` angles express `key_message`; `cta` matches
   `objective` and `funnel_stage`. Downstream work follows these fields: `objective` and
   `funnel_stage` set media optimisation and how direct the creative is; `audience` drives
   targeting and tone; every asset carries `key_message` and `cta`; each angle becomes one or more
   assets; assets and retargeting answer `primary_objection`. `objective` is one of SALES, LEADS,
   AWARENESS, TRAFFIC; `targetMetric` must fit it (SALES: ROAS, CPA, REVENUE, ORDERS; LEADS:
   LEADS, CPL; AWARENESS: REACH, CPM; TRAFFIC: CLICKS, CPC) and `targetValue` is the number reports
   judge the campaign against — ground it in past results or the brief, never invent precision.
   `business_goal` explains the commercial why. `market` and `schedule` set the geo and flight
   scope every channel inherits.
   **Budget and its split.** `plannedBudget` is the total for the flight, in the workspace
   currency (`get_account_context` → `workspace.currency`; it matches the ad account's).
   `channels[]` says how much of it each channel spends: today one entry, `META_ADS`, with its
   `plannedBudget` share (it may be all of the total) and optional `startDate` / `endDate`
   inside the schedule. The shares may not exceed the total; if the refusal says part of the total
   is already allocated to channels managed in the web app, plan within what is left. The Meta
   share averaged over its flight must stay under `maxCampaignDailyBudget`, and the channel's
   config (`channel-config`) must fit that share — ground it in the ceiling and the brief, not a
   guess.
   **What reports judge.** After launch, `targetMetric` / `targetValue` become the objective status
   (ahead / on track / behind / too early) and the shares become budget pacing in
   `get_campaign_performance`; choose them so the advertiser recognises the goal they gave you.
5. Return the draft to root for the `campaign-brand-compliance` review (Brand OS + go-back
   reason). On `revise` / `fail` → apply its `revise_notes` (correction mode) and return again.
6. Tell the advertiser in 2–4 lines what research found and what it changes, then return the draft
   or call `save_campaign_strategy` when the orchestrator allows → wait G_STRATEGY. Reopen later
   with `preview_campaign_strategy_card` (`campaignId`) — do not re-propose only to show.

## Rules

- DO NOT create a campaign, research, or draft before the alignment check passes
- DO NOT draft or propose a strategy before research (step 3) is done
- DO NOT put a claim in `reason_to_believe` or `insight` that research did not surface
- DO NOT number angles; send `supporting_messages` as `{ title, description }` in priority order
- DO NOT dump raw data rows to the advertiser; summarise findings
- DO NOT emit Meta `OUTCOME_*` objectives, placements, bid strategies, ABO/CBO budget mode,
  daily budgets, region/city-level geo targeting, or Graph dumps — `market`, `plannedBudget` and
  `channels[]` are business-level only
- DO NOT allocate budget to channels you cannot see (creators are planned by the advertiser in the
  web app)
- DO NOT invent brand facts ruled out by BINDING Brand OS
- DO NOT write assets, creatives, or channel config
- DO NOT change an approved strategy — that needs an approved Go back card first
  (`campaign-workflow`)
- DO NOT talk to the advertiser unless the orchestrator brief says you may
- DO NOT call `save_campaign_strategy` unless the brief allows (default: return draft)
- ONLY produce `CampaignStrategy` grounded in the alignment check, research, and Brand OS

## Examples

Input: Brand OS ready + goal “sell Basecamp to SMBs in RO” + new campaign  
Output: alignment check (Basecamp is offered, SMBs allowed, purchase event fires) → start-campaign
→ research (past posts/ads, audiences, past campaigns and their performance) → draft incl.
`channels: [{ type: "META_ADS", plannedBudget }]` → compliance review → summary →
`save_campaign_strategy`

Input: goal “promote our new mobile app” but Brand OS and data show no app  
Output: stop at alignment; say the app is not found in Brand OS or data and ask what to promote

Input: existing `DRAFTING` campaign, advertiser asks in chat to change the strategy  
Output: amend only contradicted fields; compliance review; save again

Input: `DRAFTING` after an approved Go back card, strategy `draft` + `needsAlignment`, reason
“lead with gifting”  
Output: amend the angles / key message / fields the change touches; keep the rest; compliance
review with `goBackReason`; `save_campaign_strategy` → G_STRATEGY

## Edge cases

- Correction mode: amend only what was contradicted; do not re-plan the rest or redo full research
- Missing Brand OS slices: refuse invention; hand off to `configure-brand-os` or list gaps
- No data sources or provider content: say so in the summary; ground on Brand OS and the brief only

## Resources

- `references/strategy-research.md` — alignment checklist, stop conditions, research questions
- `references/start-campaign.md` — read before `create_campaign` or when opening a new campaign
