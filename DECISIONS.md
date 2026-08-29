# Decision log

Significant, sticky decisions and *why* we made them — so contributors and future-us understand the
reasoning, not just the outcome. Each entry records the context, the options weighed, the decision,
and what would make us revisit it. Decisions change via pull request to this file.

Status values: **Proposed** (recommended, awaiting joint ratification) · **Accepted** · **Superseded**.

> **The Aug 2026 data-handling decisions** (leads pipeline, observation
> mirroring, paraphrase rule, canonical removal, roles, presence-as-provenance)
> live together in **[RFC-0001](rfcs/0001-data-handling.md)** — one document,
> ratified as a whole, instead of a dozen ADR entries.

---

## ADR-0001 — Data license: CC0

**Status:** Proposed (recommended) — to ratify jointly with the co-founder.

**Context.** The dataset needs a license, and the choice is *sticky*: relicensing a community dataset
later effectively requires every contributor's consent, so it's near-irreversible. Decide deliberately
once. Use the **data-native** licenses (CC0 / Open Data Commons, or CC 4.0) — pre-4.0 Creative Commons
licenses don't cleanly cover the EU *sui generis database right*; CC0 does.

**Options weighed.**

| | **CC0** / PDDL | **ODC-BY** / CC-BY-4.0 | **ODbL** / CC-BY-SA-4.0 |
|---|---|---|---|
| Demands | nothing (public domain) | attribution | attribution **+ share-alike** |
| Derivatives must stay open | no | no | **yes** |
| Adoption friction | lowest | low | higher |
| Can absorb raw OSM into the core | no | no | **yes** |
| Poster child | Wikidata | some gov open data | OpenStreetMap |

**Decision.** **CC0**, with attribution requested as a *community norm* (not a legal requirement).

**Rationale.**
- **Max adoption.** A *canonical* source only works if every consumer integrates with zero legal
  hesitation. CC0 removes all of it.
- **Matches the business model.** Money is made on apps / services / transactions built *on top* of the
  data, not on owning it — so share-alike protection isn't needed to defend a business.
- **Freshness is the moat, not the license.** A static fork goes stale/"worthless" in months, so ODbL's
  "stop private forks" protection buys little while costing real friction.

**Consequences.**
- **Cannot ingest raw OpenStreetMap** (ODbL) into the CC0 core. OSM is used only as a *lead* — piano
  locations re-derived from it — or kept in a separate, ODbL-attributed layer. (See `LICENSE`.)
- No legally-mandated attribution; we rely on the norm.
- Redistributing a *source's* data under CC0 requires that source's permission (e.g. pianos.pub, with
  Zack's explicit OK). Every observation records its `Source` + license so this stays inspectable.

**Revisit if.** OSM-derived data ever becomes *central* to the dataset — then ODbL's OSM-compatibility
might outweigh the downstream friction. Until then, CC0 wins on reach.

---

## ADR-0005 — Simplify: wiki + ambassadors instead of the event-sourced observation model

**Status:** Accepted (JB + Daniel call, 19 Aug 2026).

**Context.** The first spec revision modeled every fact as an immutable, source-attributed
observation with derived confidence resolution. Technically sound, but heavy: hard to read, hard
to explain to contributors, and more machinery than a young project needs.

**Decision.** OpenPianos works like Wikipedia. Anyone with an account edits piano records
directly; every record keeps a full attributed revision history; nothing is deleted.
**Ambassadors** (of a piano, a venue, a city, a state, or a country) review, verify, and correct
within their scope. Freshness is a first-class, human-readable signal (`lastVerifiedAt` + who),
not a computed confidence number. The first product is a simple website: sign up, add a piano,
edit, verify it's still there.

**What survives from the earlier model.** CC0; stable ids with redirect-on-merge; nothing ever
deleted; every change attributed; imports keyed to their source so re-syncs update instead of
duplicate and respect hand-curation; links-not-media; facts-not-prose.

**Consequences.** The apps carry the social layer (Plinkato: profiles, passports, plinks,
streams) and use OpenPianos as their piano source, feeding verifications back. The event-sourced
design remains in git history and can be revisited if scale or abuse ever demands it.

**Revisit if.** Vandalism or conflicting-source volume outgrows what revision history plus
ambassador review can handle.

---

## Open / upcoming decisions

To be recorded here as they're settled (several were discussed but not yet ratified):

- **ADR-0002 — Forge/hosting:** GitHub (public, community reach, neutral if co-owned) vs GitLab
  (self-hostable, sovereign; Daniel's Hyperthings instance is *private*, which conflicts with
  community-ownership). *Leaning: public community home on GitHub; engine may mirror to GitLab.*
- **ADR-0003 — Canonical store engine:** Postgres + PostGIS vs MariaDB (Daniel's brief). *Leaning:
  Postgres/PostGIS for geo + the provenance model.*
- **ADR-0004 — Venue modeling:** Piano as the atom + an optional `Venue` grouping entity for rental
  studios (vs denormalizing venue fields onto pianos). *Leaning: optional Venue entity.*
