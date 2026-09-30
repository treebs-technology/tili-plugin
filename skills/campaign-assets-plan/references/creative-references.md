# Creative references

Find the reference images each planned piece needs and attach them to that piece. The image phase
does no reference research — it renders the prompt with exactly these references (plus any the
advertiser adds on the assets card).

## Soft-skip

When the advertiser already supplied refs + clear visual direction, skip a full research pass:
upload their refs and attach them. Still encode Brand OS colors/fonts/logo rules into prompts.

## Rules

- DO NOT invent brand colors/fonts — Brand OS only
- DO NOT ignore advertiser take / don’t-take notes
- DO NOT generate finished creatives here
- DO NOT attach a reference without a `note` saying what to take (and what not to)
- Choose sources as needed: Brand OS, advertiser files, Meta, Canva, past tili media, the open web
  — tool schemas decide how to call them

## What to produce

Per piece, 1–4 prioritized references (max 8), each attached as
`visual_direction.references[] = { imageKey, note }`:

- Brand OS section images: reuse their `imageKey` directly.
- Everything else: `upload_campaign_image` (`purpose: "reference"`, public https `url`, or a file
  via presigned upload: `contentType` → PUT to `uploadUrl` → `imageKey` to verify).

Start each note with its role, then what to take and what to ignore — the image prompt mirrors it.
Roles: `product`, `logo`, `composition/layout`, `lighting`,
`palette/mood`, `styling/props`, `casting/person`, `typography layout`.

Good notes are specific: "lighting: warm window light only — ignore the model", "product: can
shape, label, colour exactly", "typography layout: headline placement only, not the colours".
Prefer a few decisive references over volume; one reference per role beats five that say the same
thing.
