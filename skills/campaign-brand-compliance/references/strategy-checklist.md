# Strategy checklist (`phase: strategy`)

Score a `CampaignStrategy` draft before `save_campaign_strategy`. Strategy is the first phase, so
alignment covers Brand OS, the draft's own consistency, and the go-back reason when present.

## Inputs to load

- `get_brand_os`: brand voice, products and pricing, guardrails / never-dos
- The `draft` and `goBackReason` from the brief
- `get_campaign_artifacts` `artifact: "strategy"` when `goBackReason` is set — the previously
  approved version shows what must stay unchanged

## Checklist

1. **`Brand OS: voice`** — pass when `big_idea`, `key_message`, `supporting_messages`, and `cta`
   read in the Brand OS voice (tone, formality, vocabulary) with no phrasing Brand OS rules out.
2. **`Brand OS: products and offers`** — pass when every product, service, price, discount, or
   offer named exists in Brand OS (or the brief's cited research) exactly as stated.
3. **`Brand OS: guardrails`** — pass when no claim, promise, audience, or topic breaks a Brand OS
   guardrail or never-do, and `reason_to_believe` makes no claim Brand OS forbids.
4. **`Alignment: strategy angles`** — pass when each `supporting_messages` angle expresses
   `key_message`, speaks to `audience` / `audience_problem`, and serves `objective`; angles do not
   overlap.
5. **`Alignment: strategy key message and CTA`** — pass when `key_message` follows from
   `proposition`, and `cta` matches `objective` and `funnel_stage`.
6. **`Alignment: strategy go-back reason`** (only with `goBackReason`) — pass when the change is
   visibly applied and fields it does not touch are unchanged from the approved version.
7. **`Alignment: market and flight`** — pass when `market` names real, sellable countries per
   Brand OS / the brief, and `schedule` is a real, non-past date range consistent with the brief.
8. **`Alignment: planned budget`** — pass when `plannedBudget` is grounded in the brief (not
   invented) and, when `schedule.endDate` is set, its daily average does not exceed the account's
   `maxCampaignDailyBudget` ceiling from `get_account_context`.

## Verdict

- `pass` — every applicable item passes.
- `revise` — one or more items fail but the draft is fixable by editing fields. Each note names
  the field and the edit: "`cta`: 'Shop now' does not fit an awareness objective — use 'Learn
  more'".
- `fail` — a required input is missing (Brand OS section, brief field) or the draft contradicts
  Brand OS at its core (wrong product, forbidden audience). Name the gap.

## Examples

- Draft promises "free shipping"; Brand OS lists no shipping offer → `Brand OS: products and
  offers` fails → `revise`: "`supporting_messages[1]`: drop free shipping; use the approved
  returns promise".
- `goBackReason: "lead with gifting"`; first angle is still "summer comfort" → `Alignment:
  strategy go-back reason` fails → `revise`: "move the gifting angle first; keep the others".
