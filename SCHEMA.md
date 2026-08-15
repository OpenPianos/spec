# OpenPianos — Data Model

The model has **three layers**. Getting this split right is the whole design: it's what lets
many sources contribute to one piano without fighting, what makes merges and deletes
survive re-imports, and what turns "stale data" into "data you can trust because you can see
where it came from and how fresh it is."

```
  Observations        →   the raw, immutable record of "what a source said, and when"
  Canonical piano     →   a stable identity resolved from its observations
  Change log          →   the append-only history of everything that happened
```

A piano is **not a row you edit.** It is a stable identity floating on a pile of immutable
observations. "Name", "status", "last verified" are *resolved* from those observations, never
overwritten. That single idea makes the rest fall into place.

---

## 1. `Piano` — the canonical entity

The resolved, public view of one real-world piano. Its scalar fields are *computed* from its
observations by the resolution rules below; nothing here is authoritative on its own.

| Field | Type | Notes |
|-------|------|-------|
| `id` | string (ULID) | **Permanent. Never changes.** The stable reference every app shares. |
| `name` | string | Resolved (highest-trust recent observation wins). |
| `latitude` / `longitude` | number | Resolved. |
| `countryCode` / `city` | string | Resolved from geocode. |
| `accessType` | enum | `public` · `bookable` · `venue` · `private` |
| `feeRequired` | bool | Free vs paid/bookable. |
| `indoor` | bool | |
| `status` | enum | `active` · `temporary` · `needs_verification` · `removed` |
| `description` | string | Optional, resolved. |
| `createdAt` / `updatedAt` / `lastVerifiedAt` | timestamp | |

**Merge = re-parenting observations.** When two canonical pianos are found to be the same
physical piano, all observations of the loser are re-pointed to the survivor and the loser is
tombstoned (`status: removed`, with a redirect). No observation is ever destroyed, so a merge
is fully reversible and can never lose a source's data.

---

## 2. `Source` and `Observation` — provenance (the crosswalk)

This is the layer most projects skip, and the reason their data rots. **Every fact about a
piano is an observation tied to the source it came from.**

### `Source`

| Field | Type | Notes |
|-------|------|-------|
| `id` | string | e.g. `pianospub`, `osm`, `ns`, `sncf`, `plinkato`, `pianomeetups` |
| `name` | string | Human label. |
| `kind` | enum | `aggregator` · `transit-operator` · `official-program` · `app` · `scrape` · `editorial` |
| `license` | string | The source's own license (drives what we may redistribute). |

### `Observation`

The immutable unit of truth: "source S said P about piano X at time T, and here's how much we
trust it." Positive observations assert existence; a `gone` observation is just a negative one.

| Field | Type | Notes |
|-------|------|-------|
| `id` | string | |
| `pianoId` | string | The canonical piano it currently attaches to. |
| `sourceId` | string | Which `Source`. |
| `sourceRef` | string | **The source's own id / URL for this record** — the crosswalk key. |
| `kind` | enum | `sighting` · `submission` · `verification` · `gone` · `moved` · `photo` · `import` |
| `observedAt` | timestamp | When it was actually observed (not when we ingested it). |
| `confidence` | number 0–1 | See the trust ladder below. |
| `lat` / `lon` | number | Where the source placed it (may differ from the canonical). |
| `payload` | json | Kind-specific detail (name, note, sightingUrl, availability, etc.). |

> **`(sourceId, sourceRef)` is unique and immutable.** It is the internal crosswalk. Re-import
> is deterministic: an incoming record whose `(sourceId, sourceRef)` already maps to a canonical
> updates *that* observation; if the canonical was merged or removed, the mapping is honored
> (fold into the survivor, or stay retired) — **so hand-curation survives every re-sync.**
> This is exactly what pianos.pub already does internally (its sightings keep an
> `originalExternalId`); OpenPianos makes it a first-class, cross-source layer.

---

## 3. Confidence — the trust ladder

Conflicting reports are resolved by **who observed, how, and how recently** — not by a naive
"latest wins." Confidence is primarily a function of the **observer**, with a boost for
proof-of-presence and a decay for age.

| Observer | confidence | |
|----------|-----------|---|
| Owner / admin (a curator's own action) | **1.00** | ground truth |
| Area ambassador, in their area | 0.90 | vetted local steward |
| Official operator (rail, airport, city program) | 0.80 | curated, current |
| Ambassador outside their area | 0.60 | trusted, not the steward there |
| Known user (history / karma) | 0.50 | a real contributor |
| Anonymous / brand-new user | 0.25 | an unverified claim |
| Scraped social sighting (e.g. Instagram) | 0.20 | + decays with age |

- **Proof-of-presence boost:** a QR scan / geo-verified check-in proves someone was physically
  at the piano — high confidence for "still here", regardless of who.
- **Recency:** effective weight ≈ `confidence × recency`. A 2019 scrape loses to a year-old
  ambassador report; a fresh official listing beats a stale delete.

### Resolution rule (precedence, not sum)

> **A lower tier never overrides a *recent* higher-tier observation.**

So an owner's "gone" cannot be undone by an anonymous "still here" or an old scrape — only by
an equal/higher tier reporting something newer. Lower tiers decide existence only where no
recent authoritative observation exists.

### Removal is a spatial + confidence claim, not a per-source flag

A deletion is a **negative observation at a location** (small radius, e.g. ~40 m) carrying its
observer's tier. On import from *any* source, a candidate piano in that radius is suppressed
only if the tombstone **out-ranks** it (higher tier, or equal tier + newer). A fresh official
listing can still revive a spot; a random scrape cannot. This is what stops one source from
resurrecting a piano another source's steward has retired — while keeping a genuinely different
piano 150 m away untouched.

---

## 4. `Photo` — an event, never a hosted image

**OpenPianos never stores image bytes.** A photo is an observation that *references* where the
image lives. This keeps the dataset portable (a single file in git), sidesteps copyright and
face-redaction entirely (the source platform owns the media), and turns every photo link into
traffic back to the app that contributed it.

| Field | Type | Notes |
|-------|------|-------|
| `id` | string | |
| `pianoId` | string | |
| `sourceId` | string | Which app/source hosts it. |
| `url` | string | **Deep link** into the source (the app screen, or the original post). |
| `thumbnailUrl` | string? | Optional, *source-hosted* thumbnail — still not ours. |
| `observedAt` | timestamp | Doubles as a freshness signal ("seen here at T"). |

Redaction, if wanted, is a *consuming app's* concern (it hosts its own media). The canonical
stays 100 % media-free.

---

## 5. `Verification`

A community confirmation that a piano is (or isn't) still there. Modeled as an `Observation`
of kind `verification`, surfaced as its own table for convenience.

| Field | Type |
|-------|------|
| `id` · `pianoId` · `sourceId` | string |
| `result` | enum `present` · `gone` · `moved` |
| `notes` | string? |
| `confidence` | number |
| `verifiedAt` | timestamp |

Verifications feed both `lastVerifiedAt` and the confidence resolution.

---

## 6. `ChangeLog` — append-only history

Every mutation writes a change record. This powers the `/changes` sync feed, the Explorer's
date scrubber (render the dataset *as of* any date), and public trust (anyone can audit how a
piano's record evolved).

| Field | Type |
|-------|------|
| `id` | string |
| `entityType` | enum `piano` · `observation` · `photo` |
| `entityId` | string |
| `action` | enum `create` · `update` · `merge` · `status_change` · `remove` |
| `actor` | string (source/user) |
| `createdAt` | timestamp |

---

## Why this beats a flat table

The failure we've all lived: you merge two "duplicates" from flattened names and later find
they were *different* pianos, or a re-sync resurrects a piano you deleted. Both come from
throwing away provenance and treating a piano as an editable row. This model fixes both by
construction: **immutable observations + a stable identity + confidence-weighted resolution +
an append-only log.** Merges become re-parenting, deletes become sticky negative observations,
and every consumer can *see* why the canonical says what it says.
