---
name: campaign-optimize
description: >-
  Live campaign optimization via MCP: read performance in three levels (campaign → channel → ad
  set), explain it against the strategy's target and budget plan, and propose changes the
  advertiser approves on a card (propose_channel_adjustment) or confirms in chat (pause / resume /
  end a channel). Always load when a channel is LIVE or the advertiser asks about performance,
  results, budget changes, or pausing. No silent spend changes.
license: proprietary
---

# Campaign Optimize

Read how a live campaign is doing, explain it in the advertiser's terms, and propose changes they
decide on.

## Reading performance

Three levels; go only as deep as the question needs.

1. **Campaign** — `get_campaign_performance` (`campaignId`): objective status (target vs actual),
   pacing against the planned budget, confidence, totals, one row per channel, data checks, and
   the platform's suggested next steps with their evidence and `action`. Start here.
2. **Channel** — `get_channel_performance` (`channelId`): KPIs with 7-day trends, pacing against
   the channel's budget share, Meta delivery stats, and one row per ad set (`adSetId`, status,
   current daily budget, spend, orders, CPA, ROAS, CTR trend, frequency).
3. **Ad set** — `get_ad_set_performance` (`channelId`, `adSetId`): its setup and one row per ad
   with the creative it runs (plan piece `assetId`, headline), to judge creatives.

How to read the numbers:

- **Sales are tracked orders** (UTM, click id, discount code) in the workspace currency.
  `reportedPurchases` / `reportedValue` are Meta's own attribution — context, never sales.
- **`confidence: LOW`** = under 10 tracked orders: do not judge ROAS / CPA / revenue or move budget
  on them; say it is too early and what to watch.
- **Settling days** (`period.settlingFrom`): orders keep arriving for ~3 days after a click; the
  last days' sales will grow. Do not call a drop from them.
- **Objective status** `AHEAD` / `ON_TRACK` / `BEHIND` / `TOO_EARLY` / `NO_TARGET` compares the
  actual with the strategy target (cumulative targets pro-rated over the flight).
- **Pacing** `OVER` / `UNDER` / `ON_PLAN` / `NO_PLAN`: spent share vs elapsed share of the flight.
- A metric with `value: null` carries `reason` — relay it, do not guess a number.
- **Checks** name data problems (tracking gaps, stale syncs, ads without UTM). A tracking
  warning comes before any optimisation advice.
- Data is synced up to yesterday. `refresh: true` on level 1 rebuilds the report (at most every
  10 minutes) — use it when the advertiser asks about today's state after a change.

## Workflow

1. Confirm which campaign (ask if ambiguous; `list_campaigns` shows a performance line for each).
2. Read level 1. Drill into a channel or an ad set only when a finding needs it.
3. Tell the advertiser in plain words: how it tracks against their target and budget plan, what
   drives it, and what is still too early to say — with the numbers the tools returned.
4. Propose at most a few concrete changes, each tied to evidence:
   - A platform suggestion fits → say which one and why; propose its `action` with
     `propose_channel_adjustment` and pass its `suggestionId`.
   - Your own finding → `propose_channel_adjustment` with the actions and a `reason` that quotes
     the numbers. Allowed: pause / resume an ad set or an ad (`level: "creative"`, Meta `adId`),
     `scale_budget` / `shift_budget` between ad sets (Meta `adSetId`s), `exclude_placement`,
     `set_age_range`. Budgets stay under the account ceiling and the channel's share.
   - The whole channel → `pause_channel` / `resume_channel` / `end_channel`, only after the
     advertiser says yes in chat (`userConfirmed: true`).
5. The adjustment card shows each change before → after; nothing reaches Meta until they approve
   it there. After approval, read level 2 again only if they ask what changed.
6. Stop — new creatives, a new audience, or a different strategy are not optimisations: hand them
   to `campaign-workflow` (a new campaign, or a go back before placement).

## Rules

- DO NOT change budgets, bids, targeting or delivery silently — every change goes through the
  adjustment card or the advertiser's explicit yes
- DO NOT accept, reject or apply a platform suggestion on the advertiser's behalf; propose it
- DO NOT judge sales-based metrics at `confidence: LOW`, or from settling days
- DO NOT present Meta-reported purchases as sales
- DO NOT invent metrics not returned by tools, or compare against targets the strategy did not set
- DO NOT discuss or change channels you do not see (creators are the advertiser's, in the web app)
- DO NOT hand-edit Meta Graph JSON — rebuilds go through `campaign-workflow` artifacts
- ONLY report, recommend, and propose changes the advertiser decides on

## Examples

Input: “How is campaign 12 doing?”  
Output: `get_campaign_performance` → “ROAS 2.4 against your target of 3 (behind), 62% of the
budget spent at 60% of the flight; Lookalike 2% costs twice the others per order” → offer the
matching suggestion as a card

Input: “Move budget from the lookalike to broad prospecting”  
Output: `get_channel_performance` (current daily budgets) → `propose_channel_adjustment`
(`shift_budget` from the lookalike `adSetId` to broad, `shiftAmount`, reason with the CPA numbers)
→ the advertiser approves on the card

Input: “Pause everything for the weekend”  
Output: confirm in chat → advertiser says yes → `pause_channel` (`userConfirmed: true`)

Input: report shows 6 tracked orders after 4 days  
Output: “Too early to judge sales (6 orders); spend is on plan, CTR is 1.3% and stable” — no budget
moves

## Edge cases

- Ambiguous campaign → ask which id before acting
- `tracking_broken` suggestion or a WARN check → fix tracking first; hold budget changes
- A change is refused on the card dry run (target not live, over the ceiling) → explain and adjust
  the proposal; do not retry the same change
- Rebuild request → hand off to `campaign-workflow` (strategy → assets → creatives → config)
