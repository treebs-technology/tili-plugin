---
name: configure-brand-os
description: >-
  Configure workspace Brand OS from official sources, user seeds, and MCP tools into the platform
  Brand OS document (logos, colors, fonts, required sections, custom rules, section image refs).
  Always load for Brand OS readiness or status questions (configured / ready / complete / Brand OS
  status), when brand is not ready, before campaign tools, or when the user asks to set up /
  configure / discover brand, fill gaps, or update brand data. Done when Brand OS is drafted and
  proposed for human approve. Reopen with preview_brand_os_card (editable until approved). Do not
  invent facts; do not approve yourself; omit unknowns. Never invent or crop a logo — leave logos
  empty if the official mark file is missing.
license: proprietary
---

# Configure Brand OS

Configure the platform Brand OS document used by the UI and downstream agents. Not a form filler.

Done when: `get_brand_os` → `ready: true` (`brandStatus` approved + required checks), after human
approve. You draft and propose; humans approve via card or Brand OS settings.

## Modes

Same tools; change scope only:

| Mode | When | Write behaviour |
| --- | --- | --- |
| From scratch | Empty / mostly empty Brand OS | Discover sources, fill all required areas |
| Fill gaps | Some slots empty | Write empty slots only; omit `replace` |
| Focused modification | User asks to change one area | Read that slice, evidence + `replace: true` only on those items |

## Workflow

1. `get_brand_os` (no section) — including status questions (“configured?”, “ready?”).
   Read **value previews** in `index` — not only filled flags. Then branch on situation state:
   - **`ready: true`** → report ready; stop unless the user asked to update.
   - **`brandStatus: pending_review`** (configured, awaiting human) →
     `preview_brand_os_card` so they can Approve or ask for changes in chat. Do not call
     `confirm_brand_os`.
   - **`brandStatus: draft` with any filled content** (partial) → `preview_brand_os_card`, report
     what is filled vs `missing`, and ask whether they want any changes or to fill gaps next.
   - **Empty / nothing filled** and status-only → report `missing` / `brandStatus`; stop unless
     they ask to configure.
2. Ask once for useful seeds when discovery is needed (prefer website URL + **official logo file**).
3. Discover from seeds, pages, and Meta as needed — choose tools from the MCP schema; do not invent
   facts from thin air.
4. Evaluate existing Brand OS before any write (empty vs filled vs custom sections). For every
   nonempty item you might change, read the full slice and compare to sources. Never treat
   `filled: true` as “content is correct.”
5. Configure areas. Logos: see `references/logos.md`. Section visual refs: see
   `references/section-images.md`.
6. `save_brand_os` mode=draft (batch preferred). Pass `unresolved` for important gaps — do not invent.
7. `save_brand_os` mode=propose → wait for human. Do not call `confirm_brand_os`.
8. If the user asks for changes in chat → fix those items only → draft → propose again.
9. To **reopen** the Brand OS card from current workspace data (no patch): call
   `preview_brand_os_card` (no args). Not approved → human edits on the card or asks you in chat
   for changes, then Approves. Approved → read-only card with a link to settings (reopen there
   before agent or UI edits).

## Rules

- Prefer authoritative sources; unknown is better than unsupported invention. Sources decide what
  you write — they are never written into Brand OS
- **Derived, not cited:** Brand OS holds the resulting rules (voice, colours, type, guardrails,
  products) as if the brand wrote them. Never add source names, URLs, "based on", "per Meta", or
  "observed on the site". Evidence stays in your reasoning; real gaps go in `unresolved`, not in
  body text
- **Language:** write every Brand OS text (logo use, colour notes, font sizes/usage, section
  bodies, `unresolved`) in the language the user writes in chat — not the website's or sources'
  language. Required section titles and enum values stay exactly as listed
- **Roles are enums, never custom:** colour and font `role` come from the schema lists only — no
  invented, descriptive, or translated labels ("Primary navy", "Promo red", "Tipografie
  principală"). Put specifics in `note` / `sizes`. Several colours may share a role
- **Type:** one face per role. `sizes` gives px sizes and how / where to use the face, in one line
- **Off-list roles already saved** (from `get_brand_os` previews) → re-save those items with the
  closest enum role and `replace: true`, moving the old label's meaning into `note` / `sizes`
- Preserve nonempty Brand OS unless justified `replace: true`; surface conflicts in `unresolved`
- Evaluate user-added / custom sections the same way as required ones before modifying them
- **Logos (strict):** only official full marks — see `references/logos.md`. Never invent, generate,
  or promote Meta/OG/favicon/moodboard into `logos[]`. If unsure → leave empty + `unresolved`
- Logos upload: `save_brand_os` `logos[].bytesBase64` — never URL-as-logo
- Section image refs: `upload_brand_os_image` → `imageKey` on `sections[].images[]` — see
  `references/section-images.md`
- Do not approve Brand OS yourself; do not lecture that branding is required — configure it
- Keep Brand OS concise and operational

## Examples

Input: “i have configured brand OS?” / “is Brand OS ready?” + Brand OS empty  
Output: `get_brand_os` → report `ready` / `brandStatus` / `missing`; stop unless they ask to
configure next

Input: “i have configured brand OS?” + `brandStatus: pending_review` (complete, awaiting approve)  
Output: `get_brand_os` → `preview_brand_os_card` for Approve; do not confirm yourself

Input: “i have configured brand OS?” + `brandStatus: draft` with some slots filled (partial)  
Output: `get_brand_os` → `preview_brand_os_card` → report filled vs `missing` and ask if they want
any changes or to fill gaps next; do not write until they ask

Input: empty brand + website URL + user attaches logo.svg  
Output: save logo bytes to `logos[]` → draft other areas → propose

Input: empty brand + website only (no logo file; page has OG/favicon crops)  
Output: draft colors/fonts/sections from evidence; **do not** put page images in `logos[]`;
`unresolved: ["official logo file"]` → propose

Input: Brand OS has colors; user asks to add moodboard images  
Output: obtain bytes → `upload_brand_os_image` → `save_brand_os` section images (not logos) → propose

Input: user asks to fix Brand voice only  
Output: read `Brand voice` slice → evidence → `replace: true` on that section only → propose

Input: website About page shows a friendly, direct tone  
Output (Brand voice body): "Friendly, direct, second person. No jargon."  
Not: "Tone is friendly (source: website About page)."

Input: brand uses navy as main colour and red for promos  
Output: `colors: [{ hex: "#002940", role: "Primary", note: "Main background, logo and headlines; pair with white" }, { hex: "#DF1D1E", role: "Secondary", note: "Time-sensitive offers and promo strip only" }]`  
Not: `role: "Primary navy"` / `role: "Promo red"`

Input: site uses Montserrat for headings  
Output: `fonts: [{ family: "Montserrat", weights: ["700"], role: "Headlines", sizes: "48 / 32 / 24px — ad headlines and hero; 700" }]`

## Edge cases

- Brand OS `approved` → reopen in settings before writes (tools refuse). Use
  `preview_brand_os_card` for a read-only card view of the approved document.
- Adding images to a nonempty section → `replace: true` and re-send body (merge skips nonempty)
- Discovery image candidates → section refs or research only, **not** logos unless the user
  confirms that exact file is the official logo
- Existing logo looks like a crop/fake → do not treat as correct; ask for official file or mark
  `unresolved` (use `replace: true` only with a verified full mark)

## Resources

- `references/logos.md` — read before writing `logos[]`
- `references/section-images.md` — read when attaching moodboard or any section image references
