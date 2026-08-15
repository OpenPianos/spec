# OpenPianos — Governance

OpenPianos is **community infrastructure**, not a product. These principles exist so it stays
trustworthy, neutral, and useful even if any one person or app disappears.

## Principles

1. **Community-owned, not app-owned.** The canonical dataset is owned by *no single application*.
   Plinkato and PianoMeetups are **consumers** of OpenPianos, exactly like everyone else — neither
   owns it. This neutrality is what lets independent sources (pianos.pub, OSM, rail operators)
   contribute without feeding a competitor.
2. **Stable identifiers.** Every canonical piano gets a permanent, opaque `id`. Ids are never
   reused and never deleted; when two pianos merge, the retired id **redirects** to the survivor —
   so any reference an app stored keeps resolving forever and never silently points to a *different*
   piano. The id names the canonical *record*, not the physical piano, so its stability is a
   guarantee we enforce (mint-once · no-reuse · redirect-on-merge), not a fact of nature. Splits
   (undoing a bad merge) are rare, logged, and surfaced in the change feed.
3. **Data preservation.** Nothing is ever physically deleted. Pianos change `status`
   (`active` / `temporary` / `needs_verification` / `removed`) and the full history is kept.
4. **Provenance & transparency.** Every fact is an attributed observation; every change is logged.
   Anyone can inspect where a record came from and how it evolved.
5. **Portability.** The data must be usable even if OpenPianos itself vanishes — hence public
   exports + a versioned git mirror. No lock-in, ever.

## Structure: two entities, one clean line

To let the founders sustain the work *without* compromising the open data:

- **A non-profit foundation** owns the neutral dataset (grant-eligible, trusted, uncapturable).
- **For-profit products** build on top — Plinkato, PianoMeetups, and any shared services (e.g. a
  studio-booking layer). These may earn freely.

> The line that must never blur: **the raw data stays free; money is made on apps, services,
> transactions, and sponsorship built on top of it.** Paywalling the data would destroy the trust
> that makes it valuable. This is the OpenStreetMap model — the map is free; an industry is built
> on it.

## How it's run (lightweight now, formal later)

- **Now:** a co-owned GitHub org (`OpenPianos`) with founders as equal Owners, a shared data
  license (CC0), and a short founders' understanding. That is enough to be real and neutral.
- **Later, when there's a reason (grants, money, partners):** incorporate the foundation
  (a Dutch *stichting*, a Spanish *asociación*, or a neutral EU structure), and move the org +
  domains from personal accounts to the foundation.

## Infrastructure neutrality

The canonical store and the public dataset must not live solely on any one party's proprietary
infrastructure. The dataset mirror lives in the **neutral public `OpenPianos` GitHub org**;
the canonical store runs on portable, self-hostable infra (Postgres/PostGIS, dumpable to a
plain file). If "community-owned" isn't backed by portable infra, it's just a word.

## Decision-making

Founders decide by consensus while the project is small. Substantive changes to this spec,
the license, or the governance model are proposed as pull requests to this repo, so the
reasoning is public and versioned.
