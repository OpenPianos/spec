# OpenPianos — Data Model

The model has **three layers**, and one organizing principle. Getting these right is the whole
design: it's what lets many sources contribute to one piano without fighting, what makes merges
and deletes survive re-imports, and what turns "stale data" into "data you can trust because you
can see where every fact came from and how fresh it is."

```
  Observations   →  the raw, immutable record of "what a source said, and when"
  Canonical piano →  a stable identity resolved from its observations
  Change log      →  the append-only history of everything that happened
```

**A piano is not a row you edit.** It is a stable identity floating on a pile of immutable
observations. "Name", "status", "last verified" are *resolved* from those observations, never
overwritten.

### It's an append-only event log — think git

Every observation (and every merge/distinct decision) is an **immutable event that lives
forever.** Nothing is ever edited or deleted; new information is always a new event. This is
event-sourcing, and the git analogy is nearly 1:1:

| git | OpenPianos |
|---|---|
| commit (immutable) | observation / assertion |
| commit history | the change log |
| **merge commit** (references, never rewrites history) | a merge assertion |
| working tree (a projection of HEAD) | the canonical `Piano` |
| `git checkout <date>` | the Explorer's date scrubber |
| `git revert` (a new commit) | undoing a bad merge (a new assertion) |
| rebuild the tree from the object graph | rebuild the canonical DB by replaying events |

Two consequences that define the whole system:

- **There is no destructive merge of pianos** — only *accumulating events* and *merging
  information* in the derived view. Even piano-membership (which observations form which canonical
  piano) is expressed as immutable `attach` / `merge` / `distinct` assertions and **derived by
  replay** — never a mutable field you rewrite.
- **The relational tables below (`Piano`, etc.) are a materialized projection** — a cache, like
  git's checked-out working tree — kept current as events arrive and **fully rebuildable from the
  event log at any time.** Events are the source of truth; the query tables are a read model. This
  is precisely why the exports, the `/changes` feed, and the GitHub mirror all fall out naturally:
  the whole database is a log you can snapshot and replay.

**The organizing principle for third-party content:**

> The canonical never holds raw third-party **media or prose**. It holds **structured,
> redistributable signal + a deep link back to the source** for the rest.

Photos → a deep link. Comments → model-extracted structured signal + a deep link. Same rule.
This keeps the dataset legally clean, portable (no blobs/prose bloating the file), and
high-signal.

---

## 1. `Piano` — the canonical entity

The resolved, public view of one real-world piano. Its scalar fields are *computed* from its
observations by the resolution rules below; nothing here is authoritative on its own.

| Field | Type | Notes |
|-------|------|-------|
| `id` | string (ULID) | **Permanent. Never changes.** The stable reference every app shares. |
| `name` | string | Resolved. |
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
tombstoned (`status: removed`, with a redirect). No observation is destroyed, so a merge is
fully reversible and can never lose a source's data.

---

## 2. `Source`

| Field | Type | Notes |
|-------|------|-------|
| `id` | string | e.g. `pianospub`, `osm`, `ns`, `sncf`, `plinkato`, `pianomeetups` |
| `name` | string | Human label. |
| `kind` | enum | `aggregator` · `transit-operator` · `official-program` · `app` · `scrape` · `editorial` |
| `license` | string | The source's own license (drives what may be redistributed). |

### A source contributes observations; it doesn't own pianos

A piano has **no single source.** Each *observation* has a source; the piano is the shared canonical
identity, owned by no one. So one canonical piano can carry observations from many sources at once —
a pianos.pub import *and* a Plinkato user's comment *and* an NS listing — and that's the whole point
of the union: the piano gets richer and fresher because many apps contribute to one identity.

- **"View by source" filters *observations*, not pianos** — and there are two distinct senses:
  - **Coverage**: "which pianos has source X observed" (the piano appears if it has ≥1 X observation).
  - **Faithful projection**: "the dataset *as X provides it*" = resolve the canonical using **only X's
    observations.** A Plinkato comment is a `plinkato` observation, so it enriches the *union* view and
    the *plinkato* view but is **absent from the pianos.pub projection** — that projection faithfully
    reconstructs what pianos.pub itself holds, never silently polluted by another source. The canonical
    is a projection; you can project it through any source filter (one source, the union, or any blend)
    from the same event log.
- Useful per-source labels are *derived*, never ownership: **discovered-by** (source of the earliest
  observation), **contributing-sources** (the set), **freshest-per-source**.
- **Provenance is sticky — no laundering, no loops.** A source only ever writes observations it
  *originated*, tagged with its own id + deep link; it never re-attributes an observation it merely
  *consumed*. So a Plinkato comment is forever `(plinkato, …)` and the pianos.pub sighting is forever
  `(pianospub, …)` — distinct `(sourceId, sourceRef)` keys, no double-counting, no feedback loop when
  data round-trips through consuming apps.

---

## 3. `Observation` — the atom of truth (provenance = the crosswalk)

This is the layer most projects skip, and the reason their data rots. **Every fact about a piano
is an observation tied to the source it came from, and to *how* we came to know it.** Positive
observations assert existence; a `gone` observation is just a negative one.

| Field | Type | Notes |
|-------|------|-------|
| `id` | string | |
| `pianoId` | string | The canonical piano it currently attaches to. |
| `sourceId` | string | Which `Source`. |
| `sourceRef` | string | **The source's own id / URL for this record** — the crosswalk key & deep link back. |
| `kind` | enum | `sighting` · `submission` · `verification` · `gone` · `moved` · `photo` · `condition` · `access` · `import` |
| `observedAt` | timestamp | When it was *observed* (not when ingested). |
| `actorRole` | enum | `owner` · `ambassador` · `official` · `known_user` · `anonymous` · `scrape` |
| `actorId` | string? | Who (optional) — lets a consumer apply its own per-user trust. |
| `presenceProven` | bool | A QR/geo-verified check-in — physical proof someone was there. |
| `method` | enum | **`reported`** (source stated it directly) · **`inferred`** (a model derived it from prose/media). |
| `derivedBy` | object? | When `inferred`: `{ model, version }` — so it's reproducible and re-extractable later. |
| `lat` / `lon` | number | Where the source placed it (may differ from the canonical). |
| `payload` | json | Kind-specific detail (name, note, url, availability, condition score…). |

> **`(sourceId, sourceRef)` is unique and immutable.** It is the internal crosswalk. Re-import is
> deterministic: an incoming record whose `(sourceId, sourceRef)` already maps to a canonical
> updates *that* observation; if the canonical was merged or removed, the mapping is honored
> (fold into the survivor, or stay retired) — **so hand-curation survives every re-sync.**
> pianos.pub already does this internally (its sightings keep an `originalExternalId`); OpenPianos
> makes it a first-class, cross-source layer.

Everything above is an **objective fact.** Note especially `actorRole`, `presenceProven`, and
`method`: they record *who* observed, whether presence was *proven*, and whether a human
*reported* it or a model *inferred* it. None of that is an opinion. The opinion — how much to
*trust* each — lives in the next section, deliberately kept separate.

---

## 4. Facts vs. confidence (the number is derived, not stored)

**Confidence is not intrinsic to the data.** It is a *policy* — an opinion about how much to trust
each origin — and different consumers can legitimately disagree. What's intrinsic is the
**provenance** (§3). Confidence is a **transparent, versioned *function* of those facts** that
OpenPianos publishes as a **default reference**, so there is a shared canonical answer — but any
consumer may recompute it with their own weights, because they have the raw facts.

So `confidence` appears in the API/exports as a **derived reference score**, clearly labelled as
such — never an opaque number someone typed in. Plinkato may trust its own signed-in users more; a
research consumer may down-weight every scrape; both are valid, because the facts are all present.

### The reference trust ladder (OpenPianos' default weighting)

| Observer (`actorRole`) | reference confidence | |
|----------|-----------|---|
| `owner` (a curator's own action) | 1.00 | ground truth |
| `ambassador`, in their area | 0.90 | vetted local steward |
| `official` (rail, airport, city program) | 0.80 | curated, current |
| `ambassador`, outside their area | 0.60 | trusted, not the steward there |
| `known_user` (history / karma) | 0.50 | a real contributor |
| `anonymous` / brand-new | 0.25 | an unverified claim |
| `scrape` (social sighting) | 0.20 | + decays with age |

Modifiers, also derived from facts:
- **`presenceProven: true`** → strong boost for "still here", regardless of who.
- **`method: inferred`** → a discount (model uncertainty stacks on top of the observer's own tier).
- **Recency** → effective weight ≈ `confidence × recency`. A 2019 scrape loses to a year-old
  ambassador report; a fresh official listing beats a stale delete.

> The "Joris = 1.0" case dissolves: nothing hard-codes the founder. It falls out of
> `actorRole: owner` under the reference model. Another consumer could weight `owner` differently —
> the *fact* ("owner did this") stays true regardless.

### Resolution rule — precedence, not sum

> **A lower tier never overrides a *recent* higher-tier observation.**

An owner's "gone" can't be undone by an anonymous "still here" or an old scrape — only by an
equal/higher tier reporting something newer. Lower tiers decide existence only where no recent
authoritative observation exists.

### Removal is a spatial + confidence claim, not a per-source flag

A deletion is a **negative observation at a location** (small radius, ~40 m) carrying its
observer's tier. On import from *any* source, a candidate piano in that radius is suppressed only
if the tombstone **out-ranks** it (higher tier, or equal tier + newer). A fresh official listing
can still revive a spot; a random scrape cannot — while a genuinely different piano 150 m away is
untouched.

---

## 5. Deriving signal from prose & media (comments, captions, photos)

Third-party **prose and media never enter the canonical raw.** They enter as (a) a deep-link
reference and/or (b) **model-derived structured observations** (`method: inferred`).

### Photos → a deep-link event

A photo is an observation of kind `photo` that *references* where the image lives. No bytes stored.

| Field | Notes |
|-------|------|
| `url` | **deep link** into the source (app screen or original post) |
| `thumbnailUrl?` | optional, *source-hosted* thumbnail — still not ours |
| `observedAt` | doubles as a freshness signal |

Face redaction never applies to the canonical (we don't host); it's a *consuming app's* concern.

### Comments → model-extracted structured observations

A free-text comment is read by a cheap model and emitted as **zero or more** structured
observations (`method: inferred`, `derivedBy: {model, version}`), with `sourceRef` = the comment
id (the deep link back). **The prose itself is never stored.**

| A comment saying… | → observation |
|---|---|
| "it's gone / removed / not there anymore" | `verification: gone` |
| "played it today, lovely" | `verification: present` |
| "badly out of tune, sticky keys" | `condition` (score) |
| "ask reception for the key, locked after 6pm" | `access` (note) |
| "beautiful Yamaha grand" | `kind` + `brand` |
| "it's actually on the other side of the square" | `moved` → **mislocation flag, never auto-moves** |

Multilingual falls out for free (the model reads Dutch/Japanese comments a keyword matcher never
could). Storing `derivedBy` means a better model can **re-extract** every signal later — without
OpenPianos ever having stored the prose.

### Safety rule for inferred signal

> **A `method: inferred` signal from a low-trust source never auto-acts.**

An anonymous comment a model reads as "gone" is a *weak negative observation* that contributes to
resolution — it does **not** unilaterally remove a piano. Prose is treacherous (*"this piano is
dead, so out of tune"* is a **condition** complaint, not a removal). The precedence rule + the
`inferred` discount are what keep extraction useful *and* safe.

### pianos.pub, mapped

Proof the model absorbs pianos.pub **losslessly** while sanitizing it:

| pianos.pub | → OpenPianos observation |
|---|---|
| **sighting** (`sourceUrl`, `observedAt`) | `sighting`, `url` = deep link, `actorRole: scrape`, `method: reported` |
| **availabilityReport** (`available` + date) | `verification: present/gone`, `method: reported` |
| **comment** (prose) | model → 0+ structured obs, `method: inferred`; **prose stays at pianos.pub** |

Each keeps `sourceRef` = pianos.pub's own id (`originalExternalId` / comment id) — the crosswalk
*and* the link back.

---

## 6. Identity resolution — "a piano at X and another very close at Y"

The hard problem. **Proximity is a signal, never an automatic decision.** Distance is ambiguous in
*both* directions: geotag drift can place the *same* piano hundreds of metres apart, while *real
distinct* pianos sit 100–270 m apart. So you cannot threshold your way to truth.

1. **Observations stay separate and immutable, whatever the outcome.** "Piano at X (source A)" and
   "piano at Y (source B)" are two observations with their own coords. Nothing is lost; every
   decision is reversible.
2. **Proximity makes them *candidates*, not a merge:**
   - **Tight distance + agreeing evidence** (same name/venue/lineage) → auto-cluster into one canonical.
   - **Fuzzy distance, or conflicting evidence** (names disagree) → **keep them as two canonicals and
     flag `needs_verification`.** Do *not* auto-merge. (This single rule prevents the Den Haag
     over-merge.)
3. **The decision is a confidence-weighted, logged action.** An ambassador/owner "these are the
   same" is authoritative and sticky; an importer's proximity guess is low-confidence and reversible.
4. **Store *both* settled calls.** A **merge redirect** *and* a **pinned-distinct assertion**
   (`distinct(P1, P2)`), so the clusterer never re-litigates a decision a human already made.

```
obs1: lat=X  name="Venestraat"    source=pianospub
obs2: lat=Y  name="Prinsegracht"  source=osm
clusterer: distance 180 m (fuzzy), names disagree → DO NOT auto-merge
  → piano P1 {obs1}, piano P2 {obs2}, both needs_verification
ambassador reads the provenance, decides distinct
  → writes distinct(P1,P2)  (logged, sticky)
next re-sync: sees the assertion → never proposes the merge again
```

The model's job is not to auto-decide the hard cases. It is to **preserve the evidence** so the
decision — human or algorithmic — is made on real provenance (names, photos, sources), not
flattened coordinates, and can always be undone.

---

## 7. Published views — canonical vs. raw

OpenPianos publishes **two views of the same log**, plus a sync feed. Both are *generated* from the
event log; neither is a separately-maintained database.

| View | What it is | For |
|------|-----------|-----|
| **Canonical / resolved** | The *sanitized* projection: one clean record per piano, conflicts resolved by the reference model, current `status`, reference `confidence`, prose reduced to structured signal, media as deep links. Exported as JSON / GeoJSON / CSV / SQLite. | Most consumers; the map/website default. |
| **Raw observations / event log** | The full provenance: every observation, every `merge`/`distinct` assertion, full history. | Auditors, the Explorer drill-down, and consumers that **re-derive their own resolution/confidence**. |
| **`/changes` feed** | Incremental change records. | Apps staying in sync. |

Two properties make the canonical *authoritative*, not just convenient:

- **Traceable** — each canonical record carries its provenance (source list + reference confidence),
  with the raw observations one hop away. Resolved ≠ black box.
- **Reproducible** — the canonical is generated by a *published, versioned* resolution model from the
  *open* event log, so **anyone can regenerate and verify it.** You hand consumers the inputs and the
  function, not just a snapshot to trust.

The website / **Explorer** defaults to the canonical resolved view and drills into the raw
observations + the date scrubber on demand — so it *observes* both: the sanitized answer up front,
the full provenance underneath.

## 8. `Verification`

A community confirmation, modeled as an `Observation` of kind `verification`, surfaced as its own
view for convenience: `result` ∈ `present` · `gone` · `moved`, plus `notes`, `verifiedAt`, and the
usual provenance facets. Feeds both `lastVerifiedAt` and the confidence resolution.

## 9. `ChangeLog` — append-only history

Every mutation writes a change record: `entityType` (`piano`/`observation`/`photo`), `entityId`,
`action` (`create`/`update`/`merge`/`distinct`/`status_change`/`remove`), `actor`, `createdAt`.
This powers the `/changes` sync feed, the Explorer's **date scrubber** (render the dataset *as of*
any date), and public trust (anyone can audit how a record evolved).

---

## Why this beats a flat table

Two failures we've all lived: you merge "duplicates" from flattened names and later find they
were *different* pianos; a re-sync resurrects a piano you deleted. Both come from throwing away
provenance and treating a piano as an editable row. This model fixes both by construction —
**immutable, attributed observations + a stable identity + facts-vs-derived confidence + sticky,
logged clustering + an append-only history.** Merges become re-parenting, deletes become sticky
negative observations, prose and media stay at their source as structured signal, and every
consumer can *see and re-weigh* why the canonical says what it says.
