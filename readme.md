# aeryx-data

Offline routing/geocoding data for [Aeryx](https://github.com/outdevnull/aeryx),
hosted as GitHub Release assets (this repo's own tracked content is
otherwise empty -- see `manifest.json` and the Releases tab).

## `manifest.json`

The worldwide region catalog the app fetches to know what's available
to download, and where. Flat per-country by default; a country can
instead carry a `subRegions` array when it's been split up -- currently
Australia (`AU`, its 7 states/territories, matching the old
`outdevnull/aeryx_old` region split) and the USA (`US`, 50 states plus
DC, generated from Natural Earth admin-1 boundaries by
[`tools/split_country.py`](https://github.com/outdevnull/aeryx/blob/main/tools/split_country.py)
in `outdevnull/aeryx`). A split country is a pure grouping node: only
its sub-regions are downloadable.

```json
{
  "version": 1,
  "generatedAt": "<ISO 8601 UTC>",
  "baseURL": "https://github.com/outdevnull/aeryx-data/releases/download",
  "countries": [
    {
      "id": "AU",
      "name": "Australia",
      "bbox": [minLon, minLat, maxLon, maxLat],
      "subRegions": [
        { "id": "au-nsw-act", "name": "...", "bbox": [...],
          "published": true, "approxSizeMB": 563,
          "tag": "osm-data", "file": "au-nsw-act",
          "updatedAt": "2026-09-21T07:14:06Z" }
      ]
    },
    {
      "id": "NZ",
      "name": "New Zealand",
      "bbox": [...],
      "published": false,
      "tag": "osm-data",
      "file": "nz",
      "updatedAt": null
    }
  ]
}
```

A downloadable entry (a flat country, or one of a split country's
`subRegions`) always carries `tag` + `file`, even when `published` is
`false` — that's the location its `.aerx`/`.aerxg` will land at once
built, so flipping `published` to `true` after a real publish is the
only change needed; no app update required to pick it up. The actual
files live at:

```
{baseURL}/{tag}/{file}.aerx
{baseURL}/{tag}/{file}.aerxg
```

`updatedAt` (ISO 8601 UTC, `null` when unpublished) is how the app
tells "downloaded and current" from "downloaded but stale, an update is
available" — it records the source data's actual last-modified time
(the newer of that region's `.aerx`/`.aerxg` GitHub Release asset
timestamps), not a manifest edit time. It only changes when the
underlying data is actually rebuilt/republished, and is bumped
automatically by `outdevnull/aeryx`'s `tools/publish_data.sh` right
after it uploads -- not a manual manifest edit. Without that bump, the
app never offers the update.

**`bbox`** is `[west, south, east, north]`. A region that crosses the
antimeridian has `west > east` (RFC 7946 §5.2's convention) -- e.g.
Alaska, `[172.476, 51.215, -129.989, 71.413]`; the app splits it into two
rectangles when downloading. Some flat, still-unpublished countries
(Russia, Fiji, ...) still carry an older ~360°-wide box instead, which
the app refuses to download until they're regenerated the same way.

**File names use the country's internet (ccTLD) code** (2026-09-27):
a whole country is just its code (`fr`, `nz`), a split country's regions
are `<code>-<region>` (`au-nsw-act`, `us-wa`). One release holds every
region's files, so the prefix is what keeps Western Australia (`au-wa`)
apart from Washington (`us-wa`), and South Australia (`au-sa`) apart
from Saudi Arabia (`sa`). It's the ccTLD, not the ISO code: the United
Kingdom's `id` is ISO `GB` but its file is `uk`. `id` stays the ISO code
(the app's own key); only `file` follows this rule.

Every country worldwide (~247, from Natural Earth admin-0 boundaries)
is listed, almost all `published: false` — there's no worldwide OSM
build pipeline yet, only Australia's 7 sub-regions have real routing/
search data (built by `outdevnull/aeryx`'s `tools/` pipeline — see
`osm-data` release). Map TILES are a separate matter: the app downloads
those live from OpenFreeMap for any region's `bbox`, published or not. Listing every country up
front, unpublished, rather than only adding an entry once it has real
data, is deliberate: the app's catalog UI should show "not yet
available" for a real place, not have no entry for it at all.

## Releases

- **`osm-data`** — `.aerx` (offline routing graph) / `.aerxg` (offline
  geocoding index) per region. Built and published by
  [`outdevnull/aeryx`](https://github.com/outdevnull/aeryx)'s `tools/`
  pipeline (ported from `outdevnull/aeryx_old` 2026-09-27): run
  `tools/sync_data.sh`, which rebuilds + publishes only when the pipeline
  or `tools/PUBLISH_DATA` changed since the last publish (bump
  `PUBLISH_DATA` to force one), and bumps `updatedAt` here afterwards.
- **`tile-regions`** — legacy: map-tile chunk boundaries (GeoJSON) built
  by `outdevnull/aeryx_old`. The current app doesn't use it (it chunks
  tile downloads from each region's `bbox` itself).
