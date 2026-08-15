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

## Access & trust — who can read vs. write

**Open to read ≠ open to overwrite.** CC0 makes the data free to read, use, and redistribute; *who may
write* is a separate question. OpenPianos is open to contribution too — but safely, because of the
observation model.

- **Contribution is open, and safe by construction.** Every write is an additive, weighted, reversible
  *observation* — nobody overwrites the canonical, they add evidence, and the confidence resolution
  decides what wins (see `SCHEMA.md §4`). An anonymous "it's gone" (0.25) cannot override a verified
  operator's "it's here" (0.95); spam is low-tier, never wins, and is reversible. **"Open write" means
  *anyone may propose an observation* — not *anyone may edit the answer*.** That's the difference from a
  wiki where one edit can blank a page.
- **Identity + trust tiers are the account system.** Contributions are weighted by *who* made them:
  `owner` → `operator` (own venue) → `ambassador` → `official` → `known_user` → `anonymous` → `scrape`.
  Anonymous is allowed but low-confidence and review-queued; contributors climb by registering, becoming
  an area ambassador, or verifying as a venue operator.
- **Apps are Sources with a key, not super-editors.** A consuming app (Plinkato, PianoMeetups) registers
  as a `Source` with an API key. That makes it *accountable* (revocable if it misbehaves) and lets it
  *attest to its own users' tiers* — but an observation's confidence reflects the **end user**, not the
  app. A plink from an anonymous app user is still anonymous-tier. No app's word is gospel.
- **Safeguards beyond confidence:** rate limits, spam detection, a review queue for low-trust
  submissions, revocable sources/accounts, and ambassador/admin moderation.
- **Transparency is the real safeguard.** Because every observation is attributed and visible in the
  Explorer — source, actor role, method, confidence, timestamp, deep link — anyone can see *who* said
  something and *how much it counts*. Open contribution is trustworthy precisely because it's paired
  with total provenance visibility.

Read is open to everyone; write is open too, but **weighted, transparent, and moderatable** — so "open"
never means "anyone can overwrite the truth."

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
