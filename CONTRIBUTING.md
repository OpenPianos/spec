# OpenPianos — Governance & contributing

How the project is run, who does what, and how data changes. (This file absorbed the former
`GOVERNANCE.md` and `DECISIONS.md` — one document for how the project works, Aug 2026.)

## Principles

1. **Community-owned, not app-owned.** No single application owns the dataset. Plinkato and
   PianoMeetups are consumers like everyone else; that neutrality is what lets independent
   sources contribute without feeding a competitor. Infrastructure runs on neutral accounts
   (hello@openpianos.net) with both founders admin.
2. **Stable identifiers.** Every piano gets a permanent id, never reused; merged ids redirect.
3. **Nothing is ever deleted.** A gone piano becomes a `removed` tombstone; its page and full
   history remain. Moderation *voids* (attributed, reasoned), never erases.
4. **Every change is attributed.** Who did what, when, is always inspectable — transparency is
   the main safeguard.
5. **Portability.** Public exports and a versioned mirror on rehostable infrastructure. The
   data must outlive the project.
6. **Founders decide by consensus** while the project is small. Substantive changes to spec,
   license, or governance land as pull requests so the reasoning stays public. When there's a
   concrete reason (grants, partners, money), incorporate a non-profit and move the org and
   domains into it.

## How data changes

**Edits** — like Wikipedia: open the record, fix it, save. Every edit writes a revision;
anything reverts. **Testimony** — the most valuable contribution is the simplest: *"I'm here,
the piano is here"* (a tap, a note, a photo). Presence signals (GPS, QR scan) are recorded as
*provenance* on a report, never required to make one. **Gone reports** are testimony too: one
report already reaches the curator inbox; status flips to `removed` only by human hand.
**Imports** — external records arrive as leads and become pianos only through a curator
decision; re-imports update rather than duplicate, and never overwrite fresher human knowledge.
The normative rules live in [RFC-0001](rfcs/0001-data-handling.md).

## Roles (as built, Aug 2026)

- **Anyone** — read; with an account (or anonymously, rate-limited): create pianos, confirm,
  note, photograph, report gone.
- **Ambassador** — a vetted person. Trust follows the person, not geography: verify, set
  photos, moderate, merge and tombstone *anywhere*; their city is their patrol responsibility
  and public identity. An ambassador's confirmation is what sets a piano's **verified badge**
  (`last_verified_at/by`) — always via their own dated, named testimony, never a bare flag.
- **Curator** — tends the dataset: works the import inbox, decides identity (create / link /
  tombstone / reject). Currently the founders plus promoted ambassadors.
- **Venue operators** (planned) — claim their venue against an official domain/phone and
  maintain their own pianos; ordinary attributed edits.

(The earlier per-piano **Keeper** role was removed Aug 2026 as too granular; it may return
with a real job if scheduled/roving pianos are built.)

## Money and neutrality

The raw data stays free (CC0), forever — paywalling it would destroy the trust that makes it
valuable. Money is made on top: apps, services, sponsorships. The OpenStreetMap model.

## Decision log

Substantive decisions and why — condensed; full reasoning in git history and RFC-0001.

- **CC0 for the data** *(accepted)*. Weighed CC0 vs ODC-BY/CC-BY vs ODbL/share-alike. CC0 wins
  on adoption (zero legal hesitation for consumers), matches the money-on-top model, and
  freshness — not the license — is the moat. Consequences: raw OSM (ODbL) can never enter the
  CC0 core (leads only, or a separate attributed layer); redistributing a source's data needs
  that source's permission; attribution is a norm, not a demand. Revisit only if OSM-derived
  data becomes central.
- **Wiki + ambassadors instead of the event-sourced model** *(accepted, 19 Aug 2026)*. The
  first spec modeled every fact as an immutable observation with confidence resolution —
  sound but heavy. Kept from it: CC0, stable ids, nothing deleted, everything attributed,
  imports keyed to source, links-not-media for external content. The observation *stream*
  returned in slimmer form as the testimony ledger (RFC-0001 §3).
- **Aug 2026 data-handling decisions** — leads pipeline, sighting mirroring,
  quote-with-attribution, canonical removal, roles, presence-as-provenance — live together in
  **[RFC-0001](rfcs/0001-data-handling.md)**, ratified as a whole.
- **GitHub as the home** *(settled by practice, Aug 2026)*: public community reach, org
  co-owned by both founders; site private until launch, spec + dataset public.
- **Postgres as the canonical store** *(settled by practice, Aug 2026)*: Neon Postgres,
  co-owned org.
- **Venue modeling** *(open)*: piano as the atom; an optional Venue grouping entity for
  bookable studios remains the leaning.

## Changing the spec

Propose changes as pull requests to this repo. RFCs (in `rfcs/`) carry substantive data-handling
changes; merge = ratification by both founders.
