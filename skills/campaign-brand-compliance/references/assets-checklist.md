# Assets checklist (`phase: assets`)

Score a `CampaignAssets` plan before `save_campaign_assets`. Every piece is checked against Brand
OS, the approved strategy, and the prompt doctrine the image generator depends on.

## Inputs to load

- `get_brand_os`: brand voice, moodboard take / don't-take, colours, type, logos (`use` notes),
  never-dos
- `get_campaign_artifacts` `artifact: "strategy"` — must be `approved`
- The `draft` and `goBackReason` from the brief
- Prompt self-check in `campaign-assets-plan/references/image-prompt.md`

## Checklist

Score every piece. Name the piece id in each failing note.

1. **`Alignment: strategy angle`** — pass when each piece serves an approved angle, the
   `key_message`, an approved audience, and only offers the strategy makes.
2. **`Alignment: strategy CTA`** — pass when each piece's `cta` fits the strategy `objective` and
   `funnel_stage`.
3. **`Brand OS: copy voice`** — pass when `hook`, `headline`, and `caption` follow the Brand OS
   voice and claims rules.
4. **`Brand OS: visual direction`** — pass when `visual_direction` follows moodboard take /
   don't-take, uses Brand OS colours and type roles, and names the logo variant its `use` note
   allows (or no logo).
5. **`Brand OS: prompt quality`** — pass when `image_generation_prompt` meets the image-prompt
   self-check:
   - structure in order: result, scene, subject and action, key details, constraints
   - every attached reference named by index with its role, matching its `note`
   - product and logo lock stated when the piece shows them
   - aspect ratio stated and matching `format`
   - on-image text rule: either quoted copy or explicit negative space — never both
   - exclusions line present (no watermark, extra logos, other brands, extra text, never-dos)
6. **`Alignment: assets references`** — pass when every reference has an `imageKey` (no raw URLs)
   and a `note` saying what to take and what not to.
7. **`Alignment: assets ids`** — pass when ids are unique and, on a re-proposal, unchanged for
   the same piece.
8. **`Alignment: assets go-back reason`** (only with `goBackReason`) — pass when the change is
   applied and untouched pieces keep copy, prompt, references, and image fields.

## Verdict

- `pass` — every applicable item passes on every piece.
- `revise` — fixable by editing named pieces. Note format: "`03` prompt: ref 2 not named — add
  'Image 2 = lighting and palette only'".
- `fail` — strategy not `approved`, required Brand OS section missing, or the plan has no pieces.

## Examples

- Piece 02 prompt has quoted headline **and** "leave the top third empty" → `Brand OS: prompt
  quality` fails → `revise`: "`02`: keep the quoted headline, drop the negative-space line".
- Piece 04 visual direction uses a neon green not in Brand OS → `Brand OS: visual direction`
  fails → `revise`: "`04`: replace neon green with accent #EC3013".
