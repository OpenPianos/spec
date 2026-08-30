# OpenPianos — Source Registry

This is the living registry of sources for the OpenPianos dataset — and, per RFC-0001, the
**licensing resolver**: an observation or photo tagged `source = X` has its terms resolved by X's
row here. The **License / Redistribution** columns are the load-bearing ones: the OpenPianos
dataset is CC0, but a source's own license (ODbL, proprietary, none stated) **cannot be
overridden** — those columns say, per source, whether its data folds into the CC0 core, stays
attributed, or needs explicit permission first. A row means "assessed", not "ingested"; the
table directly below is what actually flows today. Re-verify counts/licenses before building a
new importer.

---

## ✅ Ingested (live pipelines, Aug 2026)

| Source | Basis | Mechanism |
|---|---|---|
| **pianos.pub** | **Explicit permission from Zack Scholl, 1 Aug 2026** ("Certainly you can have my permission… periodically download the geojson"). Contact: his personal gmail + the pianos.pub domain address (the old `zack@mail.pianos.travel` bounces/goes stale — do not use). | Authorised feed snapshot (4 Aug) as baseline; feed has 401'd since ~10 Aug (his bot-limiter, presumably) so daily catchup runs via the robots.txt-advertised sitemap + per-page JSON-LD/HTML sightings. Cron 05:40 UTC. Sightings mirror as observations; captions held verbatim, displayed as attributed quotes, never exported (RFC-0001 §4). |
| **NS (Dutch Rail) station listing** | Official public listing | Complete-pull listing diff (still-listed = weak alive signal; delisting = absence event). |
| **airportpianos.org** | CC0 dedication (data; photo confirmation pending) | Listing importer. |
| **OpenStreetMap `amenity=piano`** | ODbL — leads only, never folded into the CC0 core | Overpass pulls + removal-changeset-comment harvesting for negative signals. |
| **YouTube (Data API)** | Facts about public videos via the official API; videos embedded, never copied | Daily search sweep (11 queries/languages), AI venue extraction, Nominatim geocoding; every video its own lead, no merging. Cron 05:20 UTC. |

---

## ⭐ Ingest first

The sources below are the only ones in this registry that are **genuinely open** — a stated open
license (CC0/CC-BY/ODbL) *and* an actual API or export mechanism, not just a webpage to scrape.
Everything else in this registry is either unlicensed, ToS-restricted, or low-value/redundant.
Start here.

| Source | Why it's first |
|---|---|
| **[OpenStreetMap `amenity=piano`](https://taginfo.openstreetmap.org/tags/amenity=piano) (via Overpass API)** | The only worldwide, systematically queryable source in this registry. ODbL — usable and redistributable now, but any derived records must carry OSM attribution and can't be silently relabeled CC0. |
| **[SNCF Open Data — Service d'attente en gare](https://ressources.data.sncf.com/explore/dataset/service-d-attente-en-gare-api/)** | Real structured dataset, API + CSV/JSON/GeoJSON export, 69 confirmed French stations with "Piano en libre service." ODbL — attribute + share-alike, not CC0-clean as-is. |
| **[Ville de Montréal — Pianos publics](https://montreal.ca/sujets/pianos-publics)** | 21 named, city-confirmed outdoor pianos. **CC-BY 4.0** — the cleanest license in this whole registry (a real open-data license, not ODbL's copyleft). |
| **[OpenStreetMap wiki — Tag:amenity=piano](https://wiki.openstreetmap.org/wiki/Tag:amenity=piano)** | The tag's own spec/definition page — pairs with the Overpass query above; same ODbL data underneath. |
| **[OSM Piano IDF / FR / AR (uMap)](https://umap.openstreetmap.fr/en/map/osm-piano-idf_654399)** | Three regional viewers (Île-de-France, France, Arabic-speaking countries) of the same open OSM tag — useful as a cross-check, but query Overpass directly rather than scraping the uMap UI. |
| **[Wikimedia Commons — Category:Street pianos](https://commons.wikimedia.org/wiki/Category:Street_pianos)** | Small (~43 photos, ~14 country subcats) but the structured-data layer is CC0 and permission is open — minor supplementary photo/geotag source. |

Two much larger sources — **pianos.pub** (~10,000+, 125+ countries) and **worldpianos.org /
Sing for Hope** (~6,652) — are almost certainly the biggest single databases that exist, but
neither is "genuinely open": pianos.pub's Terms explicitly forbid scraping/bulk retrieval, and
worldpianos.org states no data license at all. Both are worth an outreach email before anything
else (see Licensing flags, below) — they don't belong in an "ingest first" list, but they're the
highest-value permission asks in the whole registry.

---

## Global open databases

Worldwide in reach (even if adoption/coverage is thin) — OSM's `amenity=piano` ecosystem plus the
two largest independent aggregators.

| Source | Coverage | Access | License | Redistribution | Priority | Notes |
|---|---|---|---|---|---|---|
| [OpenPianosMap](https://github.com/brunetton/OpenPianosMap) | France-originated (SNCF, ~66 entries), extensible worldwide via OSM | Indirect, via Overpass querying the tag it defines; repo has no export | ODbL | Blocked — no license grant on the repo itself; go through OSM/Overpass instead | Low | Not a dataset — an OSM tagging campaign; its own MapContrib viewer is dead |
| [OpenStreetMap — `amenity=piano` tag (wiki)](https://wiki.openstreetmap.org/wiki/Tag:amenity=piano) | Worldwide, sparse/thin adoption (proposed-status tag) | API (Overpass/taginfo); wiki page is the spec, not a location list | ODbL (data); CC-BY-SA 2.0 (wiki text) | Open — attribute + share-alike, not CC0-clean | Medium | Best supplementary lead source; cross-check against OpenPianosMap and pianos.pub |
| [OSM Piano IDF / FR / AR (uMap)](https://umap.openstreetmap.fr/en/map/osm-piano-idf_654399) | France, Île-de-France, Arabic-speaking countries (3 regional maps) | Map-scrape (403 on direct fetch — bot-blocked); same data pullable via Overpass | ODbL | Open — attribute + share-alike | Low | Query Overpass directly rather than scraping these uMap instances |
| [Wikimedia Commons — Category:Street pianos](https://commons.wikimedia.org/wiki/Category:Street_pianos) | Worldwide, ~14 country subcats, ~43 photos total | Map-scrape (per-file, no bulk export) | CC0 (structured data layer); per-file mixed (CC-BY-SA/CC-BY/PD); page text CC-BY-SA | Open, but no unified "dataset" license — check per file | Low | Photo category, not a location registry; only a few files are geotagged |
| [MapContrib "Pianos map" (OpenPianosMap, archived)](https://www.mapcontrib.xyz/t/e5c83c-Pianos_map) | Worldwide in principle | Dead site (TLS cert mismatch / connection refused) | ODbL | Open, but nothing to fetch | Low | Defunct; use Overpass directly for the same underlying OSM data |
| [pianos.pub (Public Pianos)](https://pianos.pub/about) | Global, claims 125 countries | Feed built for us (`/pianos.geojson`) + sitemap/JSON-LD | None stated (ToS prohibits scraping — superseded for us by permission) | **Permitted** — explicit permission from Zack Scholl, 1 Aug 2026; see Ingested table above | ✅ ingested | The largest public-piano DB (~11k records); relationship maintained directly with Zack |
| [World Pianos (Sing for Hope)](https://worldpianos.org/) | Global, every continent | Map/list-only; no API or export | None stated | Unknown — site unreachable from this environment, ToS unverified | Medium (pending permission) | Likely the largest aggregate (~6,652 pianos); outreach to hello@worldpianos.org needed |

## Official open-data portals

Government/institutional datasets with an actual license and export mechanism — the highest-trust
tier in this registry.

| Source | Coverage | Access | License | Redistribution | Priority | Notes |
|---|---|---|---|---|---|---|
| [OpenStreetMap `amenity=piano` (via Taginfo)](https://taginfo.openstreetmap.org/tags/amenity=piano) | Worldwide (Europe, Asia, Americas regional breakdowns) | API (Overpass/Overpass Turbo/XAPI/ohsome); taginfo is stats/browse only | ODbL | Open — attribute + share-alike; can't be relabeled CC0 | **High** | Purpose-built tag with rich sub-tags (access, operator, opening_hours); best single lead source |
| [SNCF Open Data — Service d'attente en gare](https://ressources.data.sncf.com/explore/dataset/service-d-attente-en-gare-api/) | France — SNCF Gares & Connexions network | API + CSV/JSON/GeoJSON export | ODbL | Open — attribute + share-alike | Medium | 69 of 245 waiting-area service records are "Piano en libre service" (Yamaha B3s, SNCF×Yamaha since 2012); no lat/lon in the piano records themselves, needs a join to SNCF's station-referential dataset. Same dataset is also cross-listed via data.gouv.fr's SNCF org catalogue (which itself has no piano dataset of its own) |
| [Ville de Montréal — Pianos publics](https://montreal.ca/sujets/pianos-publics) | City of Montreal, ~10 boroughs, seasonal since 2012 | List on live page; likely also in the city's broader open "Lieux et bâtiments" export (unconfirmed for PIAP records specifically) | CC-BY 4.0 (council resolution CG14 0091) | Open — attribution note required, not pure CC0 | Medium | 21 named locations; manual pull now, follow up on the open-data export for coordinates |

## Railway & transit

| Source | Coverage | Access | License | Redistribution | Priority | Notes |
|---|---|---|---|---|---|---|
| [Grandi Stazioni — United Street Pianos Italia](https://www.grandistazioni.it) | Italy, ~6-7 major FS stations (Venice, Milano Centrale, Roma Termini, Torino, Napoli, Padova) | List-only, scattered press only — nothing on the corporate site itself | None stated | Unknown | Low | Started 2016; no structured data anywhere, only news coverage + a third-party Weebly fan page |
| [Union Station Los Angeles — Amenities](https://www.unionstationla.com/amenities/) | Single site — Passenger Concourse, LA Union Station (Metro Art "Play On") | List-only, static page | None stated | Unknown | Low | One verifiable Yamaha upright, daily 8am-8pm, 20-min limit; fine as a manually-verified single record |
| [TfL #Platform88](https://tfl.gov.uk/info-for/media/press-releases/2018/june/jamie-cullum-launches-platform88-piano) | 3 London Underground stations (rotated) | List-only (press releases; not in TfL's actual open-data/GIS hub) | None stated | Unknown | Low | Defunct 2017-2019 promo, pianos since donated to charity — not worth ingesting into a live dataset |

*(SNCF's own program is the strongest railway source and lives under Official open-data portals, above.)*

## Airports

| Source | Coverage | Access | License | Redistribution | Priority | Notes |
|---|---|---|---|---|---|---|
| [LAWA — LAX Public Pianos](https://www.lawa.org/news-releases/2019/news-release-67) | LAX, Terminal 7 + Terminal 4 Connector (2 pianos) | List-only — a single 2019 press release | None stated | **Blocked-ish** — ToS-restricted (standard press copyright) | Low | 2026 status unverified; manual-entry fodder only |
| [Dublin Airport — "public piano returns to Terminal 1"](https://www.dublinairport.com/latest-news/2025/11/02/dublin-airport-back-in-tune-as-public-piano-returns-to-terminal-1) | DUB, Terminal 1 Arrivals Hall (1 piano) | List-only — single news article | None stated | Unknown | Low | Donated 2018, restored/returned Nov 2025; one manual record |
| [Toronto Pearson — #YYKeyZ](https://www.torontopearson.com/en/while-you-are-here/art-and-culture/yykeyz) | YYZ, one piano at a time (currently Terminal 3, Gate C31) | List-only, editorial page | None stated | Unknown | Low | Already catalogued by airportpianos.org / pianos.pub / streetpianos.com — pull from those instead |
| [HKIA Arts and Culture Festival 2023](https://www.hongkongairport.com/en/media-centre/press-release/2023/pr_1682) | HKIA terminal, 3 painted pianos | List-only — single press release (direct fetch 403-blocked) | None stated | Unknown | Low | One-off festival, 28 Sep–31 Dec 2023, appears ended; no coordinates or list |
| [Artwork Archive — PIT Art in the Airport](https://www.artworkarchive.com/profile/pit-arts-and-culture) | Pittsburgh Intl Airport (PIT) only | List-only (403 on direct fetch; SaaS art-collection catalog, no API) | None stated | Unknown | Low | ~3 painted pianos inside a much larger general art collection, not a piano DB |
| [Stuck at the Airport — Airport Pianos You Can Play](https://stuckattheairport.com/2019/06/28/airport-pianos-you-can-play/) | Mostly US airports (~10-15 across the whole blog, unconsolidated) | List-only, prose blog post | Proprietary ("All rights Reserved") | **Blocked-ish** — ToS-restricted | Low | Unstructured; airportpianos.org (surfaced in search, not yet assessed) looks like a better airport-specific candidate to evaluate separately |

## National/regional directories

| Source | Coverage | Access | License | Redistribution | Priority | Notes |
|---|---|---|---|---|---|---|
| [Leeds & Bradford Piano Trail](https://www.leedspiano.com/piano-trail/) | Leeds & Bradford + Keighley, West Yorkshire, UK | List-only, prose + link-out map | None stated | Unknown | Low | Roster resets per festival edition (2021, 2024); no addresses/coordinates; needs LIPC permission before reuse |
| [MJF Piano Trail 2026](https://manchesterjazz.com/piano-trail-2026/) | Greater Manchester — city centre + Rochdale, Trafford, Salford, Oldham, Ashton, Wigan | List-only, prose | None stated | Unknown | Low | Seasonal (this edition 29 Mar–31 May 2026, already ended); 18-19 pianos, needs manual transcription + a permission ask |
| [SG Public Pianos](https://sgpianos.space/) | Singapore only (malls, MRT, public spaces) | Map-scrape; Firebase/Firestore backend, no documented public read endpoint | None stated | Unknown | Low | Crowdsourced with admin moderation; needs direct outreach to the site owner |
| [ThePiano.SG — Public Pianos](https://www.thepiano.sg/piano/public) | Singapore only | Map/list-only, WhatsApp-submitted | None stated | Unknown, leaning ToS-restricted (commercial operator, SGMusic Pte Ltd) | Low | Live site currently broken (TLS cert mismatch, redirect loop) — recheck health before any outreach |

## Art & festival programs

| Source | Coverage | Access | License | Redistribution | Priority | Notes |
|---|---|---|---|---|---|---|
| [Tokyo Street Piano Festival](https://www.tokyostreetpiano.com/) | Tokyo Station/Marunouchi/Yaesu area | List-only, plain venue list | Proprietary ("ALL RIGHTS RESERVED") | **Blocked-ish** — ToS-restricted | Low | Seasonal (~9 pianos, 2nd edition Feb-Mar 2026), not a permanent registry |
| [Tokyo Station City Management Council](http://www.tokyostationcity.com/en/) | Same festival, co-organizer's district homepage | List-only | None stated | Unknown | Low | The EN page itself has zero piano content — redundant with tokyostreetpiano.com above, kept for completeness |
| [Free the Music PGH](https://freethemusicpgh.org/) | Greater Pittsburgh | List-only, venue names only (no addresses) | None stated | Unknown | Low | ~3 painted pianos (Lucid Juice Sewickley, Gazebo @ Walcott Park, PIT airport) |
| [Chicago Park District — Pianos in the Parks](https://www.chicagoparkdistrict.com/events/pianos-parks-washington-square) | Chicago only, one page per park | List-only, per-park event pages | Proprietary | **Blocked** — ToS restricts to "personal, noncommercial use only" | Low | Seasonal (Jun 21–Jul 31 each year), 6-8 pianos/season, rotating — already on pianos.pub |
| [Seattle Parks blog / pianosintheparks.org](https://parkways.seattle.gov/?p=5520) | Seattle / King County / Bellevue / Mercer Island / Kirkland | Map-scrape (the real program is at the third-party pianosintheparks.org, not this stale 2015 city blog post) | None stated | Unknown | Low | Seasonal (~11-22 pianos depending on year); target pianosintheparks.org directly, confirm permission before scraping |

## City/tourism/university & bookable venues

| Source | Coverage | Access | License | Redistribution | Priority | Notes |
|---|---|---|---|---|---|---|
| [Piani Luimní (Limerick City & County Council)](https://www.limerick.ie/discover/living/limerick-news/piani-luimnis-first-public-piano-limerick-unveiled) | Limerick city centre, Ireland | List-only — single 2019 news article, no list/map/API | None stated | Unknown | Low | ~3 confirmed (Milk Market, Custom House, Lucky Lane); pianos.pub has a fuller crowdsourced list |

*No genuine bookable-venue (studio/practice-room) sources surfaced in this batch — a gap against
`SCHEMA.md`'s `Venue`/`accessType: bookable` scope, worth a dedicated search pass later.*

## Apps & community

| Source | Coverage | Access | License | Redistribution | Priority | Notes |
|---|---|---|---|---|---|---|
| [PianoMaps: Public Pianos (Vrexas)](https://play.google.com/store/apps/details?id=com.vrexas.pianomaps) | Unspecified/worldwide, thin | None — app-only, no website/API/export | None stated | Unknown | Low | Small niche Android app, no viable ingestion path |
| [PianoSpot / GoPianos (Measify)](https://pianospot.app/) | Global, claims 180+ cities, ~2,400+ pianos | Map/list-only; no API/export | None stated | **Blocked-ish** — ToS-restricted | Low | Crowdsourced data monetized via in-app purchase; low priority |
| [Urban Pianist — Street Piano Locations](https://urban-pianist.weebly.com/street-piano-locations.html) | London, UK only (~9 locations) | List-only, no map/API | None stated | Unknown | Low | Single-author blog, narrative only, no coordinates |
| [FlyerTalk — "Airports with public pianos" thread](https://www.flyertalk.com/forum/travelbuzz/2114503-airports-public-pianos.html) | Global, US-heavy (~10 airport mentions) | List-only, forum thread | None stated | **Blocked** — Internet Brands ToS bars automated scraping | Low | Unverified, anecdotal; manual lead-list only |
| [SecretLDN — Secret Street Pianos in London](https://secretldn.com/secret-street-pianos-never-knew-existed-london/) | London only (~5-6 spots) | List-only, static article | None stated | Unknown | Low | Lifestyle listicle, no coordinates; cross-check leads only |

---

## Licensing flags

**ODbL — cannot be relicensed CC0; usable only as an attributed, share-alike layer or as a lead
source for manual/attributed entries:**
OpenPianosMap (github) · OpenStreetMap `amenity=piano` (wiki + taginfo/Overpass) · OSM Piano
IDF/FR/AR (uMap) · MapContrib (archived) · SNCF Open Data — Service d'attente en gare.
→ Per `LICENSE`, treat OSM-derived data as a **lead/discovery layer**, not a bulk CC0 import,
unless each record is independently re-verified and re-licensed.

**Needs explicit permission before any ingestion (ToS forbids scraping, or all-rights-reserved
copyright with no reuse grant):**
pianos.pub (ToS explicitly bars "systematically retrieving data... to create... a database" —
contact zack@mail.pianos.travel) · PianoSpot/GoPianos (Measify) · Tokyo Street Piano Festival
("ALL RIGHTS RESERVED") · Chicago Park District ("personal, noncommercial use only") · Stuck at
the Airport ("All rights Reserved") · FlyerTalk (Internet Brands ToS bars automated
scraping/reuse) · LAWA LAX press release (standard press copyright).
→ These are exactly the sources worth an outreach email — several (pianos.pub, worldpianos.org)
are also the highest-value single databases in the registry.

**Cleanly open — real license, no permission blocker:**
Ville de Montréal — Pianos publics (CC-BY 4.0) · Wikimedia Commons structured-data layer (CC0,
though individual files are mixed-licensed).

**None stated / unknown (the majority of this registry):**
Every remaining airport, city-program, art-program, and app/community source above has "none
stated" for license and "unknown" for permission — treat these as **manual-entry leads**, not
importable feeds, until someone asks the operator directly. Do not bulk-scrape a source with no
stated license just because it lacks an explicit prohibition.

---

## Unverified / to-check

- **OSM Piano IDF/FR/AR (uMap)** — direct WebFetch returned HTTP 403 (bot-blocked); assessment
  leans on the OSM wiki + taginfo secondary sources, not a direct look at the uMap pages.
- **World Pianos (Sing for Hope)** — site was unreachable from this environment (repeated
  ECONNREFUSED); ToS/Privacy Policy text (updated Jan 2025) not directly confirmed, no open-data
  language surfaced in search.
- **HKIA Arts and Culture Festival 2023 press release** — direct WebFetch 403-blocked; confirmed
  only via web search snippets, not the primary page.
- **Artwork Archive — PIT profile** — direct fetch returned HTTP 403; no API/export/open-data
  endpoint confirmed either way.
- **ThePiano.SG** — live URL currently broken (TLS cert mismatch to `paradise.sg` + redirect loop
  to `/install.php`); re-verify site health before any outreach.
- **SNCF Open Data dataset** — OpenDataSoft page rendered live record counts as template
  placeholders in the crawled snapshot; the 69-station "Piano en libre service" figure and the
  ODbL license text should be re-confirmed directly against the dataset's own metadata/API before
  building an importer.
- **SG Public Pianos** — Firebase/Firestore-backed SPA with a client-side API key embedded but no
  documented public read endpoint; unclear whether that key can legitimately be used for read
  access, needs a direct question to the site owner either way.
- **Grandi Stazioni — United Street Pianos Italia** — no page exists on grandistazioni.it itself;
  current status of the ~6-7 stations (2016-2018 installs) is unverified, sourced only from
  scattered press and a third-party fan site.
- **Toronto Pearson, Dublin Airport, LAX** — single-instant-in-time press releases/news pages;
  whether these installations are still physically present has not been re-checked since original
  publication.
- **airportpianos.org** — surfaced only as a mention inside the Stuck at the Airport notes, never
  independently assessed; worth a dedicated pass, it may outrank several airport sources above.
