# Image prompt

Write one `image_generation_prompt` per planned piece. It is the only instruction the image
generator gets besides the attached references, and the advertiser cannot edit it — so it must be
complete, specific, and unambiguous on its own. Stay within 1200 characters; cut adjectives before
cutting structure. Works for any image model.

## Structure (in this order)

1. **Result** — intended use, placement, medium, aspect ratio: "Meta feed ad, 4:5 portrait, real
   lifestyle photograph".
2. **Scene** — where and when (place, season, time of day), materials and surfaces.
3. **Subject + action** — who or what, doing what; one focal point.
4. **Key details** — framing and camera, lighting, palette, texture, on-image text, reference
   roles.
5. **Constraints** — one short line: product and logo lock, exclusions, Brand OS never-dos.

Write a short narrative paragraph, not a keyword list. For a complex piece, use labeled sentences
in the same order ("Scene: … Subject: … Text: … Refs: … Constraints: …").

## Visual direction → prompt

The prompt is the executable form of the piece's `visual_direction` — no new ideas beyond it.

| Source | Becomes |
| --- | --- |
| `concept` | the single focal idea |
| `subject` | subject and action |
| `composition` | shot, angle, framing, placement in frame |
| `photography_style` | medium, lighting, finish |
| `text_overlay` | quoted copy **or** negative space |
| piece `format` | aspect ratio + safe zones |

Brand OS inputs to encode: palette hex by role, type roles, moodboard take / don't-take, the logo
variant from its `use` note, never-dos.

## Practices

- **Visible detail over mood words.** Name materials, light source, direction, and colour
  temperature, the visual medium, framing, and texture. Say "photorealistic" or "real photograph"
  when that is the goal. Camera terms ("50mm, shallow depth of field") are appearance cues, not
  physics. "Linen shirt, sand colour" beats "summer vibes".
- **People.** Give body framing, scale, gaze, and interaction with the product: "seen from the
  chest up, looking at the glass, hands naturally holding the can, label facing camera".
- **Exact text.** Quote the copy, give its position and Brand OS font role, say "exactly once",
  spell brand names, and add "no other text". Otherwise ask for explicit negative space ("clean
  empty area top third") and keep the platform safe zone free. Never both.
- **Indexed reference roles.** Name each attached reference by position with its role and what to
  ignore, mirroring its `note`: "Image 1 = product (keep shape, label, colour exactly); Image 2 =
  lighting and palette only, ignore the person and props". Say how they combine ("place the can
  from Image 1 in the lighting of Image 2"). Roles: `product`, `logo`, `composition/layout`,
  `lighting`, `palette/mood`, `styling/props`, `casting/person`, `typography layout`.
- **Describe what should be there.** "Clean empty counter" beats "no clutter". Keep exclusions to
  the one constraint line at the end.
- **Preserve lock + exclusions line.** Product geometry, label, and colour unchanged; logo as
  supplied (variant from its `use` note); no watermark, no extra logos, no other brands, no extra
  text; Brand OS never-dos.
- **Always state the aspect ratio.** Some models adopt the aspect ratio of the last reference
  image, so the explicit ratio is required.
- **Priority when sources conflict:** product ref → Brand OS → advertiser notes → mood / style
  refs. The Brand OS palette overrides a reference's palette.

## Anti-patterns

- Quality filler: "stunning, 8K, high quality, on-brand"
- "Like ref 2" without naming which aspect to take
- Several focal points, or conflicting descriptors ("minimal" + "busy market scene")
- Unquoted copy, or copy plus a negative-space request
- A reference attached but never named in the prompt

## Worked example

Visual direction (4:5 feed):

- `concept`: cold brew as the calm start to a summer morning
- `subject`: woman in her 30s pouring Ember cold brew into a glass
- `composition`: medium close-up, eye level, subject left third, headline top right
- `photography_style`: natural lifestyle photography, soft window light, warm, fine grain
- `text_overlay`: headline "Cold brew, warm mornings" top right
- References: 1 `product` — "can shape, label, colour exactly"; 2 `lighting` + `palette/mood` —
  "warm window light only; ignore the model and props"; 3 `composition/layout` — "subject left,
  headline top right; not the colours"
- Brand OS: background #F4EFE6, accent #EC3013, text #1E1E1E, display font for headlines; no
  logo on lifestyle pieces

Prompt:

> Meta feed ad, 4:5 portrait, real lifestyle photograph. A sunlit kitchen counter on an early
> summer morning, pale oak and linen. A woman in her 30s, chest up at eye level, pours Ember cold
> brew from the can into a tall glass; hands hold the can naturally, label facing camera. Subject
> in the left third, 50mm look, shallow depth of field. Soft window light from the right, warm
> tone, fine grain. Background #F4EFE6, accent #EC3013 on the straw only. Headline "Cold brew,
> warm mornings" exactly once, top right, Brand OS display font, #1E1E1E; no other text. Keep the
> bottom 15% free of detail. Image 1 = product: keep the can's shape, label, and colour exactly.
> Image 2 = lighting and palette only; ignore the person and props. Image 3 = layout only. Place
> the can from Image 1 in the light of Image 2. Can unchanged; no logo, no watermark, no other
> brands, no extra text.

## Self-check before save

The `campaign-brand-compliance` assets review scores these; fix them first.

- Could a stranger render this with only the prompt + refs and get the approved look?
- Structure in order: result, scene, subject and action, key details, constraints.
- Every attached reference is named by index with its role, matching its `note`.
- Product and logo lock stated when the piece shows them; palette by Brand OS hex.
- Aspect ratio stated and matching the format.
- On-image text: quoted copy (exactly once, no other text) **or** negative space — never both.
- Exclusions line present.

After render, the images review checks the result against this prompt: text accurate and
spelled right, product and label intact, every reference's take honoured and its don't-take
absent.
