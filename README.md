# Podcast Radar

Find podcasts made near you, or about places near you, using the Podcasting 2.0 `<podcast:location>` RSS tag.

A single static `index.html` (no build step, no backend) running entirely in the browser. Mobile-first: designed for the phone in your pocket, works fine on desktop too.

**Live at [radar.rss.io](https://radar.rss.io).**

> **Heads up: this is a proof of concept / prototype.** A companion experiment to [Podroll Atlas](https://github.com/albertobeta/podroll-atlas), this time exploring the location side of Podcasting 2.0. Rough edges expected. Issues and pull requests welcome.

## What it does

Two views, one switch:

- **Near me**: tap "Find podcasts near me", grant location access (or paste coordinates in any common format), pick a radius (km or miles, preselected from your browser locale), and get a distance-sorted list of podcasts that declare a location in their RSS feed. Tap any show for its RSS feed, Podcast Index page, a map link, and what its place actually is according to OpenStreetMap. If nothing falls inside your radius, the nearest shows beyond it are listed instead.
- **Globe**: every located podcast on a 3D globe · dots clustered by place, cover art appearing as you zoom in, overlapping covers merging into tappable stacks, and a bottom sheet listing each place's shows. An active search draws a radar ping over your position.

Both views filter by the tag's `rel` attribute (All · Made in · About), and both are shareable: the URL fragment (`#lat,lon,radius` plus an optional `,globe`) encodes the view, so a link reproduces it.

## The dataset

Source: [podcastindex.org/datasets](https://podcastindex.org/datasets), specifically the `locations.json` file published by the Podcast Index project. It aggregates every channel-level `<podcast:location>` tag found across the RSS feeds they index.

The dataset is young and growing fast (a couple hundred feeds when this app was built, thousands within days). Roughly two thirds to three quarters of rows carry usable coordinates; the rest declare only a free-text place name and are skipped, honestly counted.

Each row carries `feedId`, `podcastGuid`, `medium`, `title`, `url` (RSS), `image`, `locale` (free-text place name), `osm` (OpenStreetMap identifier), and `latlon`. The app normalizes tolerantly: it accepts `lat,lon` strings in several formats, skips rows without coordinates, and also accepts a proposed future `locations[]` array shape carrying per-place attributes.

### About `<podcast:location>`

`<podcast:location>` is one of the Podcasting 2.0 namespace tags ([spec](https://podcasting2.org/docs/podcast-namespace/tags/location)). It lets a podcaster declare where their show is made (`rel="creator"`) or what place it is about (`rel="subject"`), with a human-readable name, `geo:` coordinates, and an OpenStreetMap identifier. The app shipped with the "Made in / About" filter dormant, wired to activate the day the dataset carried `rel` · which it did, and the filter switched itself on.

## Caching

The dataset is fetched once and stored in the browser's **IndexedDB** for an hour (it regenerates server-side frequently), so repeat visits open instantly without hammering the public endpoint; a tab resumed from the background refreshes itself once the hour has passed. A cache-age indicator and a "Force refresh" button (same as Podroll Atlas) let you bypass the cache on demand. The fallback chain also mirrors Atlas: live endpoint, then a `fallback-locations.json` snapshot committed to this repo, then whatever is still in IndexedDB even if stale.

## Running it

Open `index.html` in a browser. That is the entire instruction.

Geolocation requires a secure context (HTTPS or localhost), so for local development serve the folder (`python3 -m http.server`) rather than opening the file directly, or use the manual coordinates input.

## Files

```
index.html                 The whole app.
fallback-locations.json    Dataset snapshot, served if the live endpoint fails.
404.html                   Real 404 page (also switches Cloudflare Pages out of SPA-fallback mode).
favicon.svg                Radar logo.
vendor/                    Self-hosted globe view assets (globe.gl + country outlines); third-party, licensed per vendor/LICENSES.md.
README.md                  This file.
```

## License

Released under [**Creative Commons Attribution 4.0 (CC BY 4.0)**](https://creativecommons.org/licenses/by/4.0/). You're free to use, adapt, remix and redistribute, including commercially, as long as you give appropriate credit and link back to this repository. The third-party files under `vendor/` are not covered by CC BY 4.0; they keep their upstream licences (MIT, Apache 2.0, public domain · see [vendor/LICENSES.md](vendor/LICENSES.md)).

## Credits

- Built by [**Alberto Betella**](https://betella.net). Disclosure: coded with heavy AI assistance. Architecture, design and decisions mine, most of the code AI-generated.
- Data: [Podcast Index](https://podcastindex.org/) and every podcaster who wrote a `<podcast:location>` tag.
- The `<podcast:location>` tag itself: [Podcasting 2.0 namespace](https://github.com/Podcastindex-org/podcast-namespace).
- Globe view: [globe.gl](https://github.com/vasturiano/globe.gl) (MIT) by Vasco Asturiano, bundling [three.js](https://threejs.org/) (MIT) and [h3-js](https://github.com/uber/h3-js) (Apache 2.0), with country outlines from [Natural Earth](https://www.naturalearthdata.com/) (public domain). All self-hosted, no CDN.
- Place details: [OpenStreetMap](https://www.openstreetmap.org/) via the [Nominatim](https://nominatim.org/) lookup API (© OpenStreetMap contributors, ODbL).
