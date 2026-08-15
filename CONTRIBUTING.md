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

## Making changes to the spec itself

The data model, governance, and license live in this repo. Propose changes as pull requests so
the reasoning is public and versioned.
