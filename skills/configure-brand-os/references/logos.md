# Logos and marks

Use before any `save_brand_os` write to `logos[]`.

## What belongs in `logos[]`

Only **official brand marks**: full wordmark, icon, mono variant, etc. that the advertiser actually
uses as a logo. Each entry needs real image bytes + a clear `use` note (light/dark, icon vs wordmark).

## Allowed sources (in order)

1. **User-uploaded logo file** (preferred)
2. Explicit brand-kit / press / “logo download” asset that is clearly the **complete** mark
3. User confirmation that a specific file is the official logo

## Never put in `logos[]`

- Meta ad creatives, organic post images, screenshots
- Open Graph / social preview images, favicons, app icons (unless user says that *is* the logo)
- Moodboard / section reference images
- Crops, zooms, partial letters, generated or composited “looks like the logo” fakes
- Any image you are not sure is the official full mark

Those belong in section `images[]` via `upload_brand_os_image` (see `section-images.md`), or stay out.

## If you do not have an official full mark

Leave `logos[]` empty. Add `unresolved` (e.g. `official logo file`). Do **not** invent a logo to
satisfy readiness — missing logo is correct until the real file exists.

## Upload path

Pass official file bytes on `save_brand_os` `logos[]` (`bytesBase64` + `contentType` + `name` +
`use`). Never pass a URL as the logo. Never use `upload_brand_os_image` for logos (that tool is for
section references only). See the tool schema for the exact patch shape.

## Verify

After write: `get_brand_os` `section: "logos"` — preview must show the **full** mark, not a crop.
If the preview is a fragment, remove/replace with the official file or clear and leave unresolved.
