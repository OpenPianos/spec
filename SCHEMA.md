# OpenPianos — Data model

Plain-language map of the schema. The authority is `db/schema.sql` in the site
repo; the reasoning is RFC-0001 (rfcs/0001-data-handling.md). This page stays
short on purpose.

## Piano — the record (identity)

One row per physical piano. Wiki-style: edited in place, every change kept.

| Field | Meaning |
|---|---|
| `id` | Permanent opaque id. Never reused; merged ids redirect. |
| `name` / `name_en` | Local-language name (what the sign says) + English alias when they differ. |
| `lat`, `lon`, `city`, `region`, `country`, `address` | Where. |
| `venue_type` | station / airport / cafe / park / … (consumer filter). |
| `access` | `public` · `bookable` · `ask`. |
| `hours`, `instrument` | Practical facts. |
| `description` / `description_story` | Two authored paragraphs, **local-language canonical**: the place (timeless) and the piano's story (temporal). Record content — a visitor's note is an observation, a different thing. |
| `description_en` / `description_story_en` | English renderings of the pair; null when the local language IS English. |
| `video_url` | Hero video link (YouTube/TikTok) — embedded, never our bytes. |
| `status` | `active` · `temporary` · `needs_verification` · `removed` (a tombstone keeps its page and history). |
| `last_verified_at/by` | **Ambassador-verified**: set only when an admin or approved ambassador files their own dated present-testimony. Anyone else's report is an ordinary sighting. Drives the badge. |
| `fresh_score` | 0–100 freshness, recomputed on every observation write: recency (2-year half-life) × volume × evidence span, collapsed while the latest signal is an unresolved gone-report. Verification refreshes it. |

## Revisions — the record's history

Append-only snapshots of every edit: who, when, what, note. Any revision is
revertable; nothing is ever lost.

## Observations — the witness stream

What people (and attested partner apps) say about the world: `present` /
`gone` / `note` / `photo`, with method (gps/qr/photo/operator/api — provenance,
never a gate), actor, optional text/media/url, timestamps. Site plinks and
notes land here too. Append-only; moderation *voids* (hidden_by/at/reason),
never deletes. Sightings from crosswalked sources are **mirrored here as fact
rows** — type + moment + source tag + link, with the source caption held
**verbatim** and displayed only as an attributed quote (RFC-0001 §4: never
exported, never record content). The `source` column keeps testimony and
mirrors from ever mixing in exports.

## Candidates + source_events — other people's claims

Each external source record is a **candidate** (lead), verbatim payload in
`raw`, summarized into columns (sightings count, first/last seen, `fresh_score`).
Import diffs emit **source_events** — the `/inbox` queue where curators decide:
create, link, tombstone, reject (reversible when newer evidence resurfaces).

## piano_sources — the crosswalk

`(source, source_ref) → piano`, with who linked it and when. Many refs, one
piano. It's how partner ids map to ours, how re-imports update instead of
duplicate, and how a source's future signals arrive as news about OUR piano.

## Accounts & roles

`contributor` → **Ambassador** (vetted person: verify — the badge, photograph,
moderate anywhere; their city is patrol + identity) → **Curator** (the dataset:
the inbox, identity decisions). Consuming apps pass through which of their
users acted, so attribution stays about people. Roles reasoning:
`CONTRIBUTING.md`.

## Exports & API

CC0 exports: `/pianos.geojson` ships each record with its descriptions (both
languages), `freshness`, derived activity facts (`sightings`, `first_seen`,
`last_activity`), `ambassador_verified` + `last_verified`, the video link, and
provenance — **facts ship, expression stays at the source**: observation text
and photos never export. A read API, a write API for partner attestations, and
a change feed are the contract to build.
