# Section image references

Use when a Brand OS section needs visual references (especially `Social mood board`, or any
custom section with reference images). Body text alone is not an image reference list.

## Correct path (same as frontend)

1. Obtain the image **file** yourself (user file, host fetch, Meta media). Nest does **not**
   download https image URLs.
2. `upload_brand_os_image` with `contentType` → returns `imageKey` + presigned `uploadUrl`. PUT the
   file to `uploadUrl` with that `Content-Type` (the returned `curl`). Never base64.
3. `upload_brand_os_image` with that `imageKey` → a **new** verified `imageKey` + preview `imageUrl`.
   Use the returned key (the pending one is rejected everywhere). Document storage is the **key**.
4. `save_brand_os`: attach `images[].imageKey` on the target section (with `replace: true` when the
   section is already nonempty — merge skips nonempty sections without replace). Re-send body when
   replacing.
5. Verify: `get_brand_os` with `section: "<title>"` → `images[].imageKey` + signed `imageUrl`.

## Rules

- DO store refs in `sections[].images[]` with workspace `imageKey`
- DO NOT put https URLs in the section body as a substitute for `images[]`
- DO NOT pass a URL as `imageKey`
- DO NOT use `upload_brand_os_image` for logos — logos use `save_brand_os` `logos[].bytesBase64`
- Discovery / Meta / creative / OG / favicon / crop images are **not** logos — see `logos.md`

## Logos vs section refs

| Asset | Upload | Attach |
| --- | --- | --- |
| Logo | `save_brand_os` `logos[].bytesBase64` | same call sets `imageKey` |
| Section reference | `upload_brand_os_image` | `save_brand_os` `sections[].images[].imageKey` |
