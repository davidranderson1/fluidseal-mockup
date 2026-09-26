# NEXT-SESSION-HANDOFF — fluidseal-mockup

Written 2026-09-26 (afternoon session) by the SSG Website / Updates session. Read this first,
then `fluidseal-knowledge/README.md` (hub) and `infrastructure.md` (mandatory before any
Supabase / DNS / hosting change). Board: https://claude.ai/artifact/W1wn6LQ9drHZeHNSXsb6Yi

## THE ONE THING TO KNOW

**sealsonline.com/en/flabed changed its category URL scheme.** The old
`/categories/{inch|metric}/{group}[/{profile}]` and `/categories/hard-parts/{group}` paths
return 404. Live (verified by fetch status in the Claude browser pane, 2026-09-26):

| Level | Live pattern | Example |
|---|---|---|
| Group | `/categories/{group}` | `/categories/rod-wipers` |
| Sub-category | `/categories/{group}/{group}-{inch\|metric}` | `/categories/rod-wipers/rod-wipers-inch` |
| Profile | `/categories/{group}/{group}-{inch\|metric}/{profile}` | `/categories/rod-wipers/rod-wipers-inch/an` |
| Hard parts | top-level | `/categories/lock-nuts`, `/categories/weld-on-ports`, `/categories/hardened-steel-shim` |
| Bushings | `hardened-steel-bushings/hardened-steel-bushings-{inch\|metric}` | |
| Metal face seal, V-rings, bearing isolators | under `shaft-seals/` | `/categories/shaft-seals/metal-face-seal/dc`, `/categories/shaft-seals/v-rings` |
| SBB / GE bearings | `spherical-ball-bushing/spherical-ball-bushings-{inch\|metric}/{profile}` | |
| Markets | no `-and-` | `markets/oil-gas`, `food-beverages`, `pulp-paper`, `truck-bus`, `waste-remediation` |

`redirect-map.csv` in this repo = old → live, 194 rows (29 category paths + 165 profiles;
146 exact, 2 renamed, 4 merged, 2 own category, 13 "parent" = no live page). Anything else
that links into the storefront (Posters site link maps, `Fluidseal_Product_Group_URLs_9.xlsx`
in the OneDrive Website folder) still carries the old scheme — board item 16, needs David's GO.

## Where things stand

| Thing | State |
|---|---|
| Live site | **https://mockup.fluidsealab.com** — HTTPS certificate serving (verified). **Enforce HTTPS** not yet ticked (David, board item 2). |
| Repo | Public. `index.html` (39 KB, 4 embedded logos), `styles.css`, `markets.html`, `markets/mining.html`, `redirect-map.csv`, `CNAME`, 9 equipment-form PDFs (7 root, 2 in `forms/`), this file. |
| Home page | 39 category links on the live scheme, all verified 200; profile counts refreshed from the live pages; Manufacturers band = embedded Parker / Freudenberg / GGB / Fluidseal logos → about-us (default); Rod Boot → /en/flabed/express. |
| Mining page | 12 profile groups (live scheme), 4 OEM tiles each on its own kits page (Caterpillar 1,195 / Komatsu 424 / John Deere 358 / Hitachi 367 products), 9 equipment models, products band. |
| Markets index | 32 markets; 31 link out (5 slugs fixed today), Mining is the mockup's own page. |
| Supabase `flabed` | `groups` / `items` / `markets` / `market_oems` URLs on the live scheme. **`profiles` loaded: 269 rows** (live-crawled; 213 with item_id, 254 with group_id; 15 rows in flange-seals / back-up-rings / face-and-thread-seals / head-seals have no group — those categories are not on the mockup home, board item 15). Still NOT in the Data API exposed schemas (David, item 1). DML only today — logged in marion/CHANGELOG and hub CHANGELOG. |
| Blob SHAs on `main` (re-check at push time) | index.html `36da70f0…`, markets/mining.html `5369cccd…`, markets.html `1f8ec459…`, redirect-map.csv `784d3439…`. |

## Open items (board numbers)

| # | Item | Owner | Next action |
|---|---|---|---|
| 1 | `flabed` not in Supabase Data API exposed schemas | David | Dashboard → Settings → API → Exposed schemas → add `flabed` → Save |
| 2 | Enforce HTTPS | David | GitHub → repo → Settings → Pages → tick Enforce HTTPS (cert is issued) |
| 4 | Next market | David (Q1) → Claude | "Oil & Gas?" — the project files hold the EOG brochure pages, BOP / ram-element images and the DXPE / MFP / James Walker oil-and-gas catalogs; repeat the Mining pattern |
| 6 | Manufacturer / Rod Boot link targets | David (Q2) | keep the about-us / express defaults or give targets |
| 15 | 4 live categories missing from the mockup home | David (Q3) → Claude | add Flange Seals, Back-up Rings, Face & Thread Seals, Head Seals cards (data already in `flabed.profiles`) |
| 16 | Workbook + Posters link maps on the old scheme | David (Q4) → Claude | regenerate `Fluidseal_Product_Group_URLs_10.xlsx` and re-point the Posters maps from `redirect-map.csv` — touches the Posters project |
| 9 | Live pricing / stock | David | Boutik / Website Connector, not this project |

## Tool learnings this session (also in the hub CHANGELOG)

- **raw.githubusercontent.com is reachable from this Cowork sandbox** for PUBLIC repos
  (fluidseal-mockup, marion): `curl` the raw file, `git hash-object` it, compare with the API
  blob SHA — exact bytes at zero context cost, and the only way to verify a push byte-for-byte.
  The hub is private (404). sealsonline.com, github.io and dropboxusercontent.com stay blocked.
  GitHub's raw CDN can serve the previous version for a minute or two after a push — wait, then
  re-fetch with a cache-busting query string.
- **Pushing through the connector means retyping the whole file** (`create_or_update_file`
  takes full content). Keep pages small: the logos are 32-colour palette PNGs at 240 px
  (11 KB for four) — the earlier 81 KB attempt is what aborted the last session's push.
  A trailing newline is added if the original had none (1-byte SHA drift — harmless).
- **WebFetch works on the live storefront** (new-scheme pages) and returns a real 404 for
  old-scheme pages; the Claude browser pane's `javascript_tool` with same-origin `fetch()` is the
  fast way to crawl a whole category tree and check status codes in one call.
- **Local Filesystem MCP tools failed** this session ("invalid outputSchema … draft-07");
  `device_request_folder_access` + `device_bash` / `device_stage_files` on the OneDrive Website
  folder worked instead — the workbook was cloud-only (I/O error in device_bash) until
  `device_stage_files` hydrated it.
- **Google Drive `download_file_content`:** large binaries (312 KB PDF) spill to a file on disk
  and can be decoded there; small ones (28 KB PNG) land in context as base64 and cannot be
  saved without retyping — prefer the larger sibling.
- **Logo sources:** project files `Parker.png`, `Freudenberg.jpg`, `GGB.jpg`; Drive
  "Fluidseal_Icon Only_Logo.pdf/.png/.jpg" (2026-05-28).
- **Shared hub files:** other sessions write `projects.md` / `CHANGELOG.md` concurrently — re-read
  immediately before every push (a SHA collision costs a full retype).

## Repo conventions

- David commits manually via GitHub Desktop when working locally; commit message is always
  `summary`. Sessions push via the GitHub connector with descriptive messages.
- Keep every page under the 1 MB Contents-API limit — shared CSS lives in `styles.css`.
- Product images on GitHub Pages: link the live CDN (`https://cms.sealsonline.com/uploads/…`)
  directly; no CSP issue there. Published claude.ai artifacts block them — base64-embed.
- DB writes: `flabed` schema only, schema-qualified, check `list_migrations` + marion
  CHANGELOG before, log after (marion CHANGELOG + hub CHANGELOG).
- Every outbound sealsonline link must be verified 200 on the LIVE scheme before pushing.
