# OpenPianos — Specification

**The canonical, open source of truth for publicly accessible pianos worldwide.**

*This repo is the spec. Read as deep as you like — it's written in layers: one line, then 30
seconds, then the core ideas, then the full model in [`SCHEMA.md`](SCHEMA.md).*

---

## In one line

> An open, living database of the world's public and bookable pianos — **one shared dataset that
> every app can read from *and* write back to**, instead of a hundred maps that each rot on their own.

## The 30-second pitch

Public-piano data is everywhere and trustworthy nowhere: dozens of maps and apps, each with its own
copy, all drifting stale until they're worthless. OpenPianos fixes that **not by building another
map, but by building the shared foundation underneath the maps** — one open dataset, a public API,
open exports, and a contribution workflow — so Plinkato, PianoMeetups, pianos.pub, travel apps, and
anyone else can *consume and enrich the same data*. The data is free (CC0). The value isn't owning
it — it's that it stays **fresh**, because many applications keep feeding it.

> From **many sites → many databases → duplicate effort**
> to **one dataset → many applications.**

## The problem (1 minute)

Lots of people have mapped public pianos. Almost every one of those maps is now abandoned in a
corner of the web, because they all hit the same wall: **data goes stale, and stale piano data is
worthless.** A piano that was there in 2019 may be long gone; a map that can't tell you *how fresh*
its information is, or *where it came from*, can't be trusted. And because every project keeps its
own siloed copy, the same piano is entered five times, merged wrongly, deleted in one place and
alive in another. There is no shared identity, no shared history, no shared truth.

## What OpenPianos is (1 minute)

OpenPianos is **not primarily a website.** It's three things:

1. **An Open Dataset** — canonical records, published as open exports (JSON · GeoJSON · CSV · SQLite).
2. **A Public API** — the primary integration surface (`/api/v1`), with a `/changes` feed for sync.
3. **A Community Contribution Layer** — submit / verify / report-gone, with review.

Every website — including the OpenPianos *Explorer* — is just one consumer. Scope is broader than
free street pianos: it covers **public/street pianos, bookable studios & practice rooms, and venue
pianos** (the `accessType` field carries the distinction, so each app filters to what it needs).

## Why it actually works — the five ideas (the important part)

Collecting piano locations is easy. Keeping them **trustworthy** as they flow between sources and
age is the hard part — and it's where every previous attempt failed. Five design choices make it
possible:

**1. It's a git repo for pianos (event-sourced).**
Every observation is an immutable event that lives forever. Nothing is edited or deleted; new
information is a new event. "Merging" two pianos doesn't destroy anything — it's a merge *assertion*
(like a git merge commit) that re-groups, never rewrites. The clean piano you see is a **projection**
of the log — a working tree you can rebuild, or check out *as of any past date*.

**2. Provenance-first: sources contribute observations; they don't own pianos.**
Every fact is an *observation* tagged with its source, who observed it, and how. A piano has **no
single owner** — a pianos.pub sighting, an NS listing, and a Plinkato user's comment can all attach
to the *same* canonical piano and make it richer. "View by source" then faithfully reconstructs any
one source's own slice, untouched by the others — and a source only ever writes what it *originated*,
so nothing gets laundered or double-counted when data round-trips between apps.

**3. Facts are stored; trust is *derived*.**
We store the objective origin — *signed-in Plinkato user*, *anonymous pianos.pub commenter*, *the
founder*, *a QR-verified visit*. We do **not** bake in a confidence number, because how much to trust
each origin is an opinion consumers can disagree on. OpenPianos publishes a transparent, versioned
*reference* confidence model (so there's a shared default answer), but any consumer can recompute it
with their own weights — because they have the raw facts.

**4. Structured signal in, raw content out.**
The canonical never stores third-party photos or prose. A **photo is a deep link** to where the image
lives (no hosting, no copyright or face-redaction problem). A **comment is read by a model** and
turned into structured signal — *gone*, *out of tune*, *ask reception for the key* — with a link back
to the original; the prose itself stays at the source. Legal, portable, high-signal.

**5. Two views: a clean canonical, and the full raw log.**
Most consumers want the **resolved answer** (one clean record per piano) — that's the default export
and what the map shows. Auditors and apps that want to re-weigh trust get the **raw observation log**.
Both are generated from the same event log, and the canonical is *reproducible* (published resolution
model + open log) — so it's authoritative, not "trust our snapshot."

*(The full model — entities, fields, the confidence ladder, clustering, the pianos.pub mapping — is
in [`SCHEMA.md`](SCHEMA.md).)*

## How the pieces fit

| Repo | Purpose |
|------|---------|
| **`spec`** (this one) | Data model, governance, workflows, licensing |
| `dataset` | The canonical data + scheduled open exports (the public mirror) |
| `api` | The engine/server (Node · TypeScript · Fastify · Drizzle · Postgres/PostGIS) |
| `explorer` | The bare, provenance-first reference viewer (static map + date scrubber) |

## Consumers

Plinkato, PianoMeetups, pianos.pub, travel guides, mobile apps, other open-data projects — and
future community initiatives. None of them owns the data. That's the point.

## Read next (deeper)

- [`SCHEMA.md`](SCHEMA.md) — the full data model: pianos, sources, observations, confidence, clustering, photos, changes
- [`GOVERNANCE.md`](GOVERNANCE.md) — community ownership, neutrality, the non-profit + apps structure
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — how data gets in, verified, and retired
- [`SOURCES.md`](SOURCES.md) — the source registry: what to ingest, with license + access per source
- [`DECISIONS.md`](DECISIONS.md) — the decision log (why CC0; hosting; DB engine; venue modeling)
- [`LICENSE`](LICENSE) — CC0, plus the third-party (OSM / pianos.pub) notes

## License

The **data** is dedicated to the public domain under **[CC0 1.0](LICENSE)**. Attribution is requested
as a community norm, not a legal requirement. See [`LICENSE`](LICENSE) for the important note on
OpenStreetMap/third-party sources that *cannot* be relicensed as CC0.

## Origins & credits

OpenPianos began with the vision and the *OpenPianos — Product & Technical Brief (v1)* by
**Daniel Seixas** (PianoMeetups): the canonical-open-dataset thesis, the three pillars
(dataset · API · contribution layer), the `status` model, the public API shape, and the
open-exports-plus-mirror distribution — much of which this spec carries forward directly.

This spec builds on that foundation and deepens the **data model**: provenance and the crosswalk,
the event-sourced (append-only) log, facts-vs-derived confidence, and cross-source identity
resolution.

Co-founded by **Daniel Seixas** (PianoMeetups) and **JB** (Plinkato).

## Status

Early and co-founded. This spec is a living draft meant to be argued with and improved via pull
requests — the reasoning stays public and versioned.
