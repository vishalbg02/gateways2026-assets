# Gateways 2026 — Runtime Assets

Art and video for the **Gateways 2026** mobile app, served over the
[jsDelivr](https://www.jsdelivr.com/) CDN and downloaded by the app on first
launch. Keeping these out of the app binary took it from **~26 MB of bundled
assets down to well under 1 MB**, and lets artwork be updated without shipping
an app-store release.

> **Do not hand-edit this repo.** Everything here is generated. Edit the source
> art in the app repo and re-run the generator.

## How it's generated

From the app repo (`gateways2026_application`):

```bash
python3 scripts/build-assets.py --out ../gateways2026-assets
git -C ../gateways2026-assets add -A && git -C ../gateways2026-assets commit -m "..."
git -C ../gateways2026-assets push
```

The generator reads `assets/` in the app repo, re-encodes everything to WebP at
the resolution it is actually displayed at, deduplicates byte-identical files,
and writes `manifest.json`.

## Layout

```
manifest.json                          the only mutable file — the app reads this
images/events/<name>.<hash8>.webp      event badges, 512px
images/characters/<name>.<hash8>.webp  character art, 800px
images/<name>.<hash8>.webp             misc UI art
videos/<name>.<hash8>.mp4              splash video
```

## Why filenames carry a hash

The 8-character suffix is a prefix of the file's SHA-256. It is the entire
versioning scheme:

- **Changed bytes produce a new filename**, therefore a new CDN URL — so a stale
  cached copy is impossible and there are no git tags to manage.
- The app can treat *"a file with this name exists on disk"* as proof its
  contents are correct, so it never re-hashes megabytes on startup.
- Byte-identical assets collapse to a single stored file that several manifest
  keys point at.

Only `manifest.json` is mutable. jsDelivr caches branch URLs for ~12h, so a
manifest change propagates within that window (or immediately via jsDelivr's
purge endpoint).

## CDN URLs

```
https://cdn.jsdelivr.net/gh/vishalbg02/gateways2026-assets@main/manifest.json
https://cdn.jsdelivr.net/gh/vishalbg02/gateways2026-assets@main/<path from manifest>
```

## manifest.json

```jsonc
{
  "version": 1,
  "generatedAt": "2026-09-17T…",
  "baseUrl": "https://cdn.jsdelivr.net/gh/vishalbg02/gateways2026-assets@main/",
  "assets": [
    { "key": "event/promptx", "path": "images/events/promptx.a1b2c3d4.webp",
      "bytes": 64611, "sha256": "…" }
  ]
}
```

`key` is the stable name the app asks for; it never changes when the artwork does.
