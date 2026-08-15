# OpenPianos — Specification

**The canonical, open source of truth for publicly accessible pianos worldwide.**

OpenPianos is an open-data initiative, not an app. It provides a shared dataset, a
public API, open exports, and a contribution workflow — so that many applications can
consume *and* contribute to the same data instead of each maintaining its own stale copy.

> The goal: move the ecosystem from **many sites → many databases → duplicate effort**
> to **one dataset → many applications.**

This repository holds the **specification**: the data model, governance principles, the
contribution workflow, and the licensing. It is the reference other repos implement.

## The problem we're solving

Public-piano data is scattered across dozens of maps and apps, and almost all of it goes
stale — which makes it, in practice, worthless. The fix isn't another map. It's a single,
open, *living* dataset that:

- keeps a **permanent identity** for every piano,
- records **where every piece of information came from** (provenance),
- **never deletes** — it changes status and keeps history,
- weighs conflicting reports by **who observed them and how recently** (confidence),
- and is **freely reusable by anyone**, forever.

## What "publicly accessible" means (scope)

Broader than free street pianos. In scope:

- **Public / street pianos** — free, open to anyone (stations, squares, airports, malls).
- **Bookable studios & practice rooms** — libraries, music schools, rehearsal spaces you can rent.
- **Venue pianos** — hotel lobbies, cafés, cultural centres open to the public.

The `accessType` + `feeRequired` fields carry this distinction, so consumers can filter to
whatever subset they need (e.g. Plinkato shows free public pianos; a studio-booking app
shows the bookable ones).

## The three pillars

1. **Open Dataset** — the canonical records, published as open exports (JSON / GeoJSON / CSV / SQLite).
2. **Public API** — the primary integration surface (`/api/v1`), including a `/changes` feed for incremental sync.
3. **Community Contribution Layer** — submit / verify / report-removed, with review.

Every website (including the OpenPianos [Explorer](#explorer)) is *just one consumer.*

## Repository map

| Repo | Purpose |
|------|---------|
| **`spec`** (this one) | Data model, governance, workflows, licensing |
| `dataset` | The canonical data + scheduled open exports (the public mirror) |
| `api` | The engine/server (Node · TypeScript · Fastify · Drizzle · Postgres/PostGIS) |
| `explorer` | The bare, provenance-first reference viewer (static map + date scrubber) |

## Consumers

Plinkato, PianoMeetups, pianos.pub, travel guides, mobile apps, other open-data projects —
and future community initiatives. None of them owns the data. That's the point.

## Read next

- [`SCHEMA.md`](SCHEMA.md) — the data model (pianos, sources, observations, confidence, photos, changes)
- [`GOVERNANCE.md`](GOVERNANCE.md) — community ownership, neutrality, the non-profit + apps structure
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — how data gets in, verified, and retired
- [`LICENSE`](LICENSE) — CC0, plus the third-party-data notes

## License

The **data** is dedicated to the public domain under **[CC0 1.0](LICENSE)**. Attribution is
requested as a community norm, not a legal requirement. See [`LICENSE`](LICENSE) for the
important note on OpenStreetMap/third-party sources that *cannot* be relicensed as CC0.
