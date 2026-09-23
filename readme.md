# aeryx-data

Offline routing/geocoding data for [Aeryx](https://github.com/outdevnull/aeryx),
hosted as GitHub Release assets (this repo's own tracked content is
otherwise empty -- see `manifest.json` and the Releases tab).

## `manifest.json`

The worldwide region catalog the app fetches to know what's available
to download, and where. Flat per-country by default; a country can
instead carry a `subRegions` array when it's been split up (currently
just Australia, `AU`, into its 7 states/territories, matching the
old `outdevnull/aeryx_old` region split).

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
          "tag": "osm-data", "file": "nsw-act" }
      ]
    },
    {
      "id": "NZ",
      "name": "New Zealand",
      "bbox": [...],
      "published": false,
      "tag": "osm-data",
      "file": "nz"
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

Every country worldwide (~247, from Natural Earth admin-0 boundaries)
is listed, almost all `published: false` — there's no worldwide OSM
build pipeline yet, only Australia's 7 sub-regions have real data
(ported forward from the old repo's `tools/build_data.sh` pipeline,
still actively run — see `osm-data` release). Listing every country up
front, unpublished, rather than only adding an entry once it has real
data, is deliberate: the app's catalog UI should show "not yet
available" for a real place, not have no entry for it at all.

## Releases

- **`osm-data`** — `.aerx` (offline routing graph) / `.aerxg` (offline
  geocoding index) per region. Built by `tools/build_data.sh` in
  `outdevnull/aeryx_old` (not yet ported to this project).
- **`tile-regions`** — map-tile chunk boundaries (GeoJSON), same
  region split as `osm-data`.
