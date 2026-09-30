# Start campaign (open container)

Open a campaign container, then continue into strategy authorship in the same phase. Do not
stop after `create_campaign` for an empty handoff.

## Rules

- DO NOT invent brand facts missing from tools/conversation
- DO NOT write assets, creatives, or media here
- DO NOT open a second campaign for the same work without asking (continue vs create new)
- DO NOT quietly resume an existing campaign without explicit advertiser choice
- DO NOT persist a free-text brief — messaging lives in `CampaignStrategy`
- ONLY create + ground the container, then continue strategy authorship (or resume after they choose)

## Workflow

### 1. Load the account

Call `get_account_context` first. Record currency (later budgets; never convert), timezone, connected
page, pixel presence, and `maxCampaignDailyBudget` ceiling. For sales goals on a product range, also
record catalog readiness (catalog, product count, pixel link, ViewContent / AddToCart / Purchase
events) — see Catalog readiness in `strategy-research.md`.

### 2. Brand OS (must-gate)

1. `get_brand_os` (no section) — if `ready` is false, **stop**. Load `configure-brand-os`; tell the
   advertiser campaign tools stay locked until Brand OS is approved; follow that skill only.
2. When ready, load BINDING Brand OS (logos/colors/fonts + sections).
3. When needed: lasting prefs via Brand OS `preferences` / `save_account_preferences` (always
   available; not Brand-OS-locked), one kebab-case key per topic (e.g. `budget-style`,
   `creative-dont`, `markets`, `approval-habits`). Never invent prefs.

### 3. Business-fit + goal completeness

Covered by the alignment check in `strategy-research.md` (campaign-strategy step 1). Do not
create a campaign until it passes.

### 4. Providers + create

1. Before `create_campaign`: `list_campaigns` for open work. If one matches this goal, **stop and ask**:

   > You already have campaign **{name}** (id {id}), status **{status}** — {stage}.
   > 1. Continue that campaign
   > 2. Create a new campaign

   - **Continue** → `get_campaign_status` + `get_campaign_artifacts` (no `artifact`, full
     context); follow `campaign-workflow` (resume step)
   - **Create new** → proceed
2. When needed: list accessible providers / set providers while still `DRAFTING`.
3. `create_campaign` with `name` (3–80 chars) and optional `providers` (default META).

### 5. Continue into strategy

Do not stop for a separate handoff. Continue drafting `CampaignStrategy` toward
`save_campaign_strategy`. Status stays `DRAFTING` until the strategy card is proposed.

## Edge cases

- Declined proposal → amend same campaign id; do not create a duplicate
- Unfinishable campaign → `cancel_campaign` when needed; do not leave duplicates without asking
