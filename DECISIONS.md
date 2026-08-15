# Decision log

Significant, sticky decisions and *why* we made them — so contributors and future-us understand the
reasoning, not just the outcome. Each entry records the context, the options weighed, the decision,
and what would make us revisit it. Decisions change via pull request to this file.

Status values: **Proposed** (recommended, awaiting joint ratification) · **Accepted** · **Superseded**.

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

## Open / upcoming decisions

To be recorded here as they're settled (several were discussed but not yet ratified):

- **ADR-0002 — Forge/hosting:** GitHub (public, community reach, neutral if co-owned) vs GitLab
  (self-hostable, sovereign; Daniel's Hyperthings instance is *private*, which conflicts with
  community-ownership). *Leaning: public community home on GitHub; engine may mirror to GitLab.*
- **ADR-0003 — Canonical store engine:** Postgres + PostGIS vs MariaDB (Daniel's brief). *Leaning:
  Postgres/PostGIS for geo + the provenance model.*
- **ADR-0004 — Venue modeling:** Piano as the atom + an optional `Venue` grouping entity for rental
  studios (vs denormalizing venue fields onto pianos). *Leaning: optional Venue entity.*
