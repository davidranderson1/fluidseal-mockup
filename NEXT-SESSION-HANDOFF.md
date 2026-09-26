# NEXT-SESSION-HANDOFF — fluidseal-mockup

Written 2026-09-26 by the SSG Website / Updates session. Read this first, then
`fluidseal-knowledge/README.md` (hub) and `infrastructure.md` (mandatory before any
Supabase / DNS / hosting change).

## Where things stand

| Thing | State |
|---|---|
| Live site | **https://mockup.fluidsealab.com** — GitHub Pages from `main`, Azure DNS CNAME `mockup` → `davidranderson1.github.io` (zone fluidsealab.com, RG Dns-rg). HTTPS cert was still issuing on 2026-09-25; **Enforce HTTPS** not yet ticked. |
| Repo | Public (required for Pages on this plan). Files: `index.html`, `styles.css`, `markets.html`, `markets/mining.html`, `CNAME`, 9 equipment-form PDFs (7 in root, 2 in `forms/`). |
| Home page | Categories home mirroring sealsonline.com/en/flabed with the reorganised layout: O'Rings & Kits, Kits & Parts, Hard Parts, Inch, Metric, Categories, Custom Quote Forms, Manufacturers. All 42 category links verified live. |
| Markets | `markets.html` = 32-market index copied from the live site (31 link out to sealsonline; Mining is the mockup's own page). `markets/mining.html` = hero + intro from live, 12 profile groups, 4 OEMs, 9 equipment models with the forms' 5 part categories, products band. |
| Equipment forms | All 9 PDFs uploaded by David in 3 batches; Mining page links and DB point at the files where they landed. Two earlier Dropbox links were **private** (audience `no_one`) — never link Dropbox share URLs from this connector on a public page. |
| Supabase | Schema `flabed` on the shared project (`hnmbjqhxvxakhdzgetxw`). Migrations `flabed_website_catalog_v1` (sections/groups/items/profiles — 8/40/48/0) and `flabed_markets_v1` (markets/market_oems/market_equipment/market_groups — 32/4/9/17). Both logged in `marion/CHANGELOG.md`. `public`/`archive`/`xpress` untouched. **`flabed` is NOT yet in the Data API exposed schemas** — the site can't query it from the browser until David ticks it. |
| Open Items board | https://claude.ai/artifact/W1wn6LQ9drHZeHNSXsb6Yi (David's rule: anything pending goes here, never chat-only). |

## In-flight when this session ended

`index.html` update **NOT pushed** — the last push was aborted mid-call. The intended
change (built and integrity-checked locally, file was ~81 KB):

1. **Manufacturers section:** replace the 5 monogram cards with embedded base64 logos
   (Parker, Freudenberg, GGB from `/mnt/project/*.jpg|png`; Fluidseal icon rendered from
   `Fluidseal_Icon_Only_Logo.pdf` at 300 dpi, cropped, downscaled to 200 px). Total ~47 KB.
   Links: all 5 → `https://www.sealsonline.com/en/flabed/about-us` (verified page; a
   reversible default — board item 6 asks David for the real targets).
2. **Rod Boot card** (Custom Quote Forms): `href="#"` → `https://www.sealsonline.com/en/flabed/express`,
   label "Coming soon · request via Custom Seal Quote".

Current `index.html` blob SHA on `main`: `239009196a6b6021ca70e024b44dc2a34732829f`.
Rebuild = re-read index.html, splice the Manufacturers `<section>` and the Rod Boot `<a>`,
push with that SHA (re-check at push time — the shared-resource rule).

## Open items (board numbers)

| # | Item | Owner | Next action |
|---|---|---|---|
| 1 | `flabed` not in Supabase Data API exposed schemas | David | Supabase dashboard → Settings → API → Exposed schemas → tick `flabed` → Save |
| 2 | Enforce HTTPS | David | GitHub → repo → Settings → Pages → tick **Enforce HTTPS** once the cert shows issued |
| 4 | Other 31 markets are link-outs | David → Claude | David names the next market (Oil & Gas / Forestry are SSG core); Claude repeats the Mining pattern |
| 5 | Manufacturer logos | Claude | Push the in-flight `index.html` above |
| 6 | Manufacturer / Rod Boot link targets | David | Confirm or change the about-us / express defaults |
| 7 | Komatsu / John Deere / Hitachi OEM tiles link to the generic OEM Kits page | Claude | Find each OEM's kits URL on the live site (needs web_search → web_fetch; Caterpillar's is `/categories/oem-kits-parts/mobile-equipment-seal-kits/caterpillar/caterpillar-kits`) and update the 3 links + `flabed.market_oems.url` |
| 8 | `flabed.profiles` empty | Claude | Load from the Profiles tab of `Fluidseal_Product_Group_URLs_9.xlsx` (Google Drive Claude/Files) |
| 9 | Live pricing / stock | David | Boutik platform — Website Connector proposal, not this project |
| 10 | Redirect map for a live cut-over | Claude | Finding: the live site already serves accessories at `/categories/{slug}`; only `/categories/accessories` (the grouping page) would change → one redirect. Can be closed with that note. |

## Tool learnings that cost hours this session — read before repeating them

- **Chrome MCP freezes** on GitHub's React `<select>` (Pages branch picker), GitHub's
  visibility modals, and Azure's blade renderers. Every freeze = 4-minute timeout and a
  hung bridge (Filesystem MCP dies with it; Claude Desktop restart clears it). What works:
  single-step calls (never `browser_batch`), `find` → click by element `ref`, and for
  Azure navigate straight to the resource URL (`#@sealsonline.com/resource/subscriptions/…`)
  instead of clicking through the list. Screenshots time out while a modal is open —
  use `read_page`/`find` instead.
- **GitHub sudo mode:** making a repo public prompts for David's password on a
  `Confirm access` page. Stop there; he types it. Sudo then holds for hours.
- **GitHub browser uploader:** 25 MB per file AND fails on a ~70 MB single commit —
  batch it. "Try again" drops the target folder path (files land in root).
- **Binaries:** the GitHub connector cannot push, move or rename binary files
  (`create_or_update_file` is text-only; no git-tree ops). GitHub's web editor cannot
  rename PDFs either. Link to files where they land, or ask David to re-upload.
- **Dropbox connector:** `fetch` extracts text only ≤ 5 MB; `create_shared_link` makes
  private (`no_one`) links only; temp `download_link` URLs are single-use and the sandbox
  egress blocks `dropboxusercontent.com` anyway. Public links need the Dropbox web UI.
- **Sandbox egress** blocks sealsonline.com, github.io, dropboxusercontent.com — verify
  live pages in Chrome, not curl.
- **`copy_file_user_to_claude`** was available on 2026-09-25 and absent on 2026-09-26;
  `file_upload` (Chrome) no longer takes host paths in this desktop-app version.
- **Published claude.ai artifacts** block ALL external images by CSP (cms.sealsonline.com
  AND the `_next/image` proxy). GitHub Pages does not. Base64-embed for artifacts.
- **`has_pages`** in the GitHub search API lags the Settings page; trust the page text.
- **Shared hub files:** other sessions write `projects.md` / `CHANGELOG.md` concurrently —
  I hit a SHA collision mid-write. Re-read immediately before every push.

## Repo conventions

- David commits manually via GitHub Desktop when working locally; commit message is
  always `summary`. Sessions push via the GitHub connector with descriptive messages.
- Keep every page under the 1 MB Contents-API limit — shared CSS lives in `styles.css`.
- Product images on GitHub Pages: link the live CDN (`https://cms.sealsonline.com/uploads/…`)
  directly; no CSP issue there.
- DB writes: `flabed` schema only, schema-qualified, check `list_migrations` + marion
  CHANGELOG before, log after (marion CHANGELOG + hub CHANGELOG).
