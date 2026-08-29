# RFC-0001 — How OpenPianos handles data

- **Status:** Draft — for review at the founders' session, 2 Sep 2026
- **Authors:** Joris Weimar; review & co-authorship: Daniel Seixas
- **Relates to:** SCHEMA.md, GOVERNANCE.md, ADR-0005

## Summary

OpenPianos aggregates many sources but is authored by people. This RFC fixes
the rules for how outside data enters the system, what counts as a record,
what counts as evidence, and what we promise never to do. Everything below is
implemented and running; this document makes it policy.

## 1. The pipeline: sources → leads → decisions → records

1. Every external source (open data, operator lists, community maps, partner
   feeds) is ingested into a **candidate layer**: one lead per source record,
   with the source's full payload kept **verbatim and read-only** (`raw`).
   Summaries (sighting counts, date spans, names) are derived columns over it.
2. Changes between pulls become **events** in a review inbox: new lead,
   changed, fresh sighting, absence claim.
3. **Nothing becomes a piano without a human decision.** A person accepts a
   lead (create), links it to an existing piano, records a removed piano, or
   rejects it. Every decision is attributed and logged. Bulk tools execute
   many decisions at once but never invent one.
4. Re-imports are idempotent: a decided lead never returns; a linked source
   record's news arrives as events on *our* piano, not as a duplicate.

## 2. Records: the claim, with history

A **piano** is one real-world instrument with a permanent id (never reused).
Its record (name, location, venue type, access, hours, description, status)
is authored content. Every change writes a **revision** — a full snapshot,
attributed. Reverting writes an old snapshot as a *new* revision; history is
append-only and public. A removed piano keeps its record (`status: removed`):
absence is data, and tombstones prevent ghost re-adds.

## 3. Observations: the witness stream

An **observation** is what a person witnessed at the piano: *present*,
*gone*, a note, a photo — attributed, timestamped, with method. Three laws
separate observations from revisions:

1. **Fold:** revisions fold to latest-wins; observations accumulate — the
   stream is the value.
2. **Undo:** revisions revert; observations are never rewritten. A faulty
   observation is **voided** by a moderator (it stops counting, stays in the
   log, the voiding itself is attributed) — struck from consideration, not
   from the record.
3. **Trust:** only observations may set freshness (`last_verified_at`) or
   feed the confidence value. Editing a record can never make a piano look
   seen.

One human act may produce rows in both streams (create a piano + attach your
photo); they share an `act_id` so the history reads as one visit.

**Assertions vs attachments.** An observation's `type` is what the person
asserts (present / gone / note / photo); attachments are its exhibits:
`media_key` for media we host (CC0, face-blurred) or `url` for an external
link (a YouTube video, an Instagram post). No per-platform fields — how a link
embeds is presentation logic. An attached link never upgrades trust by
itself: freshness follows the asserted verdict, not the exhibit.

## 4. Source evidence is not testimony

A source's observation stream (e.g. dated third-party sightings) is kept as
**evidence on the lead** — counts, spans, kinds, full stream in `raw` — and
shown during review. It is **never imported into our observations**: no
verdict, no method, no attributable witness. Imports fill *presence*; only
people (or attested partner apps via the write API, marked with their source)
produce observations. A record created from a source is therefore *canonical
but unverified* until someone stands at the piano.

## 5. The crosswalk

`(source, source_ref) → piano` maps every external record onto our piano —
many refs, one piano (geocoding scatter routinely mints several records for
one instrument). Every crosswalk row records **who linked it and when**.
The crosswalk is what lets partners consume our data and report back against
their own ids — and what stops round-tripped data from becoming fake
corroboration: exchanged records must carry origin ids both ways, and
mirrored records are excluded from evidence counts.

## 6. Names and places

`name` is the **local name** — what the signage says — and is canonical.
`name_en` is an optional international alias (auto-filled from source data
where available) for script accessibility; venue names are transliterated,
not translated, and no per-language name matrix is kept. Country values are
normalized to English in the data; interfaces localize display.

### Photos and the feature image

Photos are **publish-first, post-moderated** — like every other contribution
(wiki model: live immediately, revertable, attributed). Ambassadors oversee
through the activity trail and may replace or remove any piano's photo;
superseded or removed photos are *voided* observations — struck from
consideration, kept in the log. The automated safety pipeline (unsafe-content
screening, face blurring) runs post-publish and can unpublish.

A piano's **feature image is derived, not stored**: the first approved,
non-voided photo observation — overridable by an explicit curator pick.
Hiding the underlying observation heals the hero automatically.

## 7. Licensing

- The dataset is **CC0, without asterisks**, achieved by the candidate layer:
  externally-licensed data stays leads; published records are human-authored.
- **Contributions are CC0 including notes and photos**: posting is dedicating.
  Uploaders affirm the photo is their own; CC0 waives copyright, not the
  privacy of bystanders — faces are blurred in served images and the
  moderation norm is *pianos, not people*.
- **Source content:** facts are extracted; prose is never republished; images
  are never copied — links only. Third-party photos cannot be laundered into
  CC0, by upload or by AI transformation (a derivative work stays the
  photographer's; a synthetic image is fabricated testimony and is rejected
  for that reason alone).
- **Links in exports:** exports may link to external *pages* as evidence
  (source records, videos); they never link to source *media files*. Media
  URLs appearing in the dataset are exclusively our own CC0-dedicated
  uploads — a media URL in OpenPianos data always means "you may use this".
  Linked external pages are not covered by the dedication.

## 8. Generated content

AI may draft record prose **from held facts only** (anti-invention rules),
into an editable field, reviewed by the publishing human; the revision is
flagged as AI-drafted. Generated content never enters observations, never
sets freshness, and is never unmarked.

## 9. What we never do

- Publish a piano no human accepted.
- Let an import or an edit set verification freshness.
- Rewrite or silently delete history — revisions or observations.
- Republish source prose or images, under any transformation.
- Ship an unattributed decision: creations, links, voids, and reverts all
  carry a name.

## Open questions for review

1. Confidence value v1: formula (recency decay + independent sources +
   human verification) and where it appears first.
2. Stale-status decay: months-without-verification before `needs_verification`.
3. Partner write API: attestation requirements for `method: api` observations.
