# OpenPianos — Contributing

Data enters, gets verified, and is retired through explicit workflows. **No data is ever
permanently deleted** — retirement is a status change plus a logged reason.

Contributions arrive two ways: **humans** (through a consuming app or the API) and **importers**
(bulk sources like pianos.pub, OSM, rail operators). Both become attributed `Observation`s.

## Piano submission

```
Submit  →  Pending review  →  Approved  →  Published
```

- A submission is an `Observation` of kind `submission` with the submitter's confidence tier.
- On approval it either **creates** a new canonical piano or **attaches** to an existing one
  (matched by `(sourceId, sourceRef)`, else by proximity + name).
- High-trust submitters (owner, area ambassador) may be auto-approved; anonymous ones queue.

## Verification

```
User visits piano  →  confirms present / gone / moved  →  stored  →  lastVerifiedAt updated
```

A verification is an `Observation` of kind `verification`. A QR scan / geo-verified visit adds a
proof-of-presence boost. Verifications feed the confidence resolution, so a fresh high-trust
"present" keeps a piano alive and a fresh high-trust "gone" begins retiring it.

## Removal

```
Report missing  →  review (weighed by tier)  →  status → removed  (history kept)
```

Removal is a **negative observation at a location**, carrying the reporter's tier (see the
confidence model in `SCHEMA.md`). It suppresses re-imports of the same spot from *any* source
that doesn't out-rank it — but a genuinely different nearby piano, or a newer higher-tier
positive report, is unaffected. Nothing is deleted; the piano is tombstoned and can revive.

## Claiming & operator self-service

The freshest, highest-trust data for a venue comes from **whoever operates it** — a library knows its
practice-room hours, that a piano was removed, that a new grand arrived. So a verified operator can
claim and maintain their own venue directly. Operator edits are just **high-confidence observations**
(`actorRole: operator`) — authoritative for their venue, but still provenance-tagged, logged, and
reversible; they never erase history or another source's attribution.

**Claim → verify → manage:**

1. **Claim** — the operator asserts they run venue *X*.
2. **Verify ownership** (rising rigor): a link to an address at the venue's **official email domain**;
   a **one-time code** to the venue's official phone / postal address; a **verification token** placed
   on the venue's official website; or ambassador/admin review for edge cases.
3. **Manage** — once verified, they update hours, booking URL, fees; add pianos; mark one removed (a
   high-confidence `gone` observation). All written as `operator` observations against the `Venue` and
   its `Piano`s.

**Access surfaces** (an institution shouldn't have to join any one app to manage its own data):

- **A neutral OpenPianos "venue portal"** — a small authenticated web form to claim + edit. The
  primary, app-independent path.
- **Any consuming app** may also surface a "claim your venue" flow and write via the contribution API.
- **Direct API with a key** — for institutional operators with their own systems (a rail operator or
  library network pushing an official list programmatically).

**Safety:** verification gates the claim; every edit is a reversible, provenance-tagged observation; a
false claim is revoked and its observations down-weighted. An operator contributes authoritative signal
about *their own* venue — they cannot delete history or override another source's provenance.

## Importers (bulk sources)

An importer maps each incoming record to an `Observation` keyed by `(sourceId, sourceRef)`:

- **already mapped, active canonical** → update that observation; re-resolve the canonical.
- **mapped to a merged/removed canonical** → honor it (fold into survivor, or stay retired).
  *This is how hand-curation survives every re-sync.*
- **unmapped** → cluster by proximity + name → attach or create.

Importers must set `Source.license` honestly (see `LICENSE`) and must never overwrite a
higher-trust human observation with a scraped one.

## Photos

Contribute a **deep link**, never an image file. A photo is an `Observation` of kind `photo`
with a `url` into the hosting app/post (see `SCHEMA.md`). The canonical stays media-free.

## Why facts, not prose (and links, not media)

OpenPianos stores *structured facts* extracted from comments — never the raw comment text — and
*deep links* to photos — never the image bytes. This isn't only about tidiness; it's deliberate,
and it holds **even when we have the source's permission to ingest**. Reasons, strongest first:

1. **Platform permission ≠ author permission.** Facts (*"out of tune", "ask reception for the key"*)
   are not copyrightable, so we can store them freely. But the *words* of a comment are the
   **author's** copyrighted expression — and a source granting us its *database* can't necessarily
   sublicense every commenter's prose for **CC0 republication**. Extracting facts removes the
   author's expression entirely, so there is no author copyright to clear.

2. **Privacy, and CC0 is irrevocable.** Comments contain names and personal detail. Publishing raw
   prose into an open, public-domain, **un-retractable** dataset would republish that PII globally,
   forever, with no way to take it back. Structured facts carry no personal content.

3. **Quality & scope.** We want piano *signal*, not a comment section — no ads, spam, abuse, or
   off-topic chatter. Extraction is the filter.

4. **Portability & reach.** Typed facts keep the dataset a small, queryable, single-file export;
   they also normalise across languages (a Dutch or Japanese comment becomes the same structured
   signal a keyword matcher never could).

5. **It's what data exchanges cover.** Partner agreements (e.g. pianos.pub) are scoped to *data*,
   not the verbatim republication of users' prose.

**Bulk ingestion still requires the source's permission/license** for the *database* itself (in the
EU, the sui-generis database right applies even to uncopyrightable facts) — but that is a separate
question from storing users' *words*, which we never do. Every observation records its `Source` and
that source's `license`, so what may be redistributed is always inspectable. When adding a new
source, confirm its license/permission first (see `LICENSE`), and — for an authoritative public
dataset — have per-source ingestion reviewed by counsel before scaling.

## Making changes to the spec itself

The data model, governance, and license live in this repo. Propose changes as pull requests so
the reasoning is public and versioned.
