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
| `description` | The authored paragraph on the piano page (record content — a visitor's note is an observation, a different thing). |
| `video_url` | Hero video link (YouTube/TikTok) — embedded, never our bytes. |
| `status` | `active` · `temporary` · `needs_verification` · `removed` (a tombstone keeps its page and history). |
| `last_verified_at/by` | Cache over observations — only a human confirmation ever sets it. |

## Revisions — the record's history

Append-only snapshots of every edit: who, when, what, note. Any revision is
revertable; nothing is ever lost.

## Observations — the witness stream

What people (and attested partner apps) say about the world: `present` /
`gone` / `note` / `photo`, with method, actor, optional text/media/url,
timestamps. Append-only; moderation *voids* (hidden_by/at/reason), never
deletes. Sightings from crosswalked sources are **mirrored here as fact rows**
(type + moment + source tag + link — never their text or media; see RFC-0001
§4), tagged by `source` so testimony and mirrors never mix in exports.

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

`contributor` → **Keeper** (tends one piano) → **Ambassador** (an area: verify,
photograph, moderate) → **Curator** (the dataset: the inbox, identity
decisions). Consuming apps pass through which of their users acted, so
attribution stays about people.

## Exports & API

CC0 exports (GeoJSON/CSV/JSON) in the dataset repo, each record carrying its
crosswalk with evidence facts (sighting counts/spans/links — facts ship,
expression stays at the source). A read API, a write API for partner
attestations, and a change feed are the contract to build.
