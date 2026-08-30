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
   **Identity is the only gate.** There is no separate "canonical" blessing:
   a piano with an identity is in the dataset, carrying computed
   reliability/freshness values; what to display is the consumer's threshold
   choice, never an editorial one (three layers: identity is human, evidence
   is accumulated and human-vetted, display is computed + user preference).
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

## 4. Source evidence: facts mirror in, expression stays out

A source's observation stream (e.g. dated third-party sightings) lives
verbatim on the **lead** (`raw`) and is shown during review. Once a lead is
crosswalked to a piano, every sighting in that stream also exists as an
**observation row on the piano** (decided 30 Aug 2026): the **facts** only —
type (present/gone), moment, source tag, and a link to the source — because
facts are not expression and ship freely, on the piano page and in the CC0
dataset. The sightings' **text and media are expression**: they stay at the
source, reachable through the link, never copied.

Mirrored rows are not testimony, and three rules keep the line honest:

1. **Never human verification.** A mirrored row never sets
   `last_verified_at` and carries no actor — the score weighs
   `source = 'pianos.pub'` differently from `source = 'site'`. A record
   created from a source stays *published but unverified* — and scored
   accordingly — until someone stands at the piano.
2. **Idempotent and reconciled.** A partial unique index (piano, source,
   moment) makes re-links and re-imports insert only what is new. If a
   source revises history (sighting deleted, spam pulled), the import diff
   flags it to the inbox ("sighting stream revised upstream") for hand
   reconciliation — sources rarely revise, so this is a tripwire, not a
   pipeline.
3. **No echo.** Outbound testimony (feeds, partner exchanges) filters
   `source = 'site'` and attested partner slugs — a mirror of their data
   never leaves us wearing our uniform.
4. **Quote with attribution, never absorb.** A mirror row carries the
   sighting's caption **verbatim** in `text` (capped at 500 chars), and is
   displayed only as an attributed quote — quotation marks, source name,
   link to the original. The `source` column is the metadata that keeps it
   honest: mirror text is someone else's expression held under ordinary
   quotation, it is **excluded from every export** (observations never ship
   in the CC0 dataset), and it never seeds record content — descriptions
   must restate the facts in our own words (§5). Ambassador-voidable, same
   post-moderation stance as photos. *(Supersedes the earlier AI-paraphrase
   rule, 30 Aug: paraphrasing a one-line eyewitness note risked distortion
   under our own voice; a labeled quote is both more honest and lossless.)*

Live attestations differ from scraped history: a partner app **pushing** a
claim through the write API stands behind it in real time and lands as a
first-class observation marked with its slug.

**Every source speaks its own dialect of evidence.** Each ingester's job is to
translate that dialect into the one shared lead shape; the loader diffs
source-agnostically. Sighting-stream sources (pianos.pub) carry explicit dated
observations per piano. **Listing sources** (NS, airportpianos, OSM) carry no
dates at all — their evidence *is the diff between pulls*:

- **Still listed** on a complete pull is a weak alive-signal, dated by the
  fetch — a maintained listing saying "there's a piano here" *today* is
  current information.
- **Delisted is absence evidence.** Silence after speech is a statement: for
  an actively maintained listing, removal is often the strongest signal that
  source ever emits. A complete pull is diffed against what we hold; records
  the source no longer lists emit an absence event ("no longer listed on X").
  This only runs on pulls known to be complete — a partial fetch must never
  mass-emit absences — and the event names the kind of absence, because
  strength varies by source (an NS delisting is strong; an OSM deletion may be
  a mapper edit, not a missing piano). Humans and the score weigh it, as with
  all source evidence.

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

**Licensing = provenance resolved through the source registry.** A photo row
records where it came from (`observations.source`); what that MEANS is looked
up in the source registry (SOURCES.md), which states each source's terms:

- `'site'` → contributed under the CC0 dedication → CC0.
- an import slug whose registry entry is CC0/public domain → adoptable as
  CC0 outright. (No current photo source qualifies: airportpianos' public-
  domain dedication covers its DATA — and awaits a path/LICENSE confirmation
  from the maintainer; see its source profile.)
- an import slug without such terms (e.g. pianos.pub) → a curator-derived
  feature photo (best sighting, faces blurred) is NOT CC0: stored with the
  original's link and an explicit marker, shown on the site, never
  distributed in the dataset — a placeholder until an ambassador's own CC0
  photo replaces it.

The license lives at the SOURCE level, never guessed per row — so when a
source's terms are ratified or change, the registry entry is the single
switch.

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
