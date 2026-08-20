# OpenPianos — Governance

OpenPianos is community infrastructure, not a product. These principles keep it trustworthy and
neutral even if any one person or app disappears.

## Principles

1. **Community-owned, not app-owned.** No single application owns the dataset. Plinkato and
   PianoMeetups are consumers like everyone else. This neutrality is what lets independent
   sources contribute without feeding a competitor.
2. **Stable identifiers.** Every piano gets a permanent id. Ids are never reused; when two
   records merge, the retired id redirects to the survivor, so stored references keep resolving.
3. **Nothing is ever deleted.** Like Wikipedia, every record keeps its full edit history. A gone
   piano changes status to `removed`; its page and history remain.
4. **Every edit is attributed.** Who changed what, and when, is always visible. Transparency is
   the main safeguard: because anyone can inspect the history, bad edits are easy to spot and
   easy to revert.
5. **Portability.** The data must outlive the project: public exports and a versioned mirror,
   on infrastructure that anyone could rehost. No lock-in.

## Roles

- **Anyone** can read, and anyone with an account can edit.
- **Ambassadors** adopt a scope (one piano, a venue, a city, a state, or a country) and keep it
  accurate: they review recent edits, verify pianos in person, and correct mistakes. Ambassadors
  are appointed simply while the project is small (founders' judgment); a formal process can come
  when scale demands one.
- **Venue operators** (a station, library, hotel) can claim their own venue and maintain its
  pianos directly; a claim is verified against the venue's official email domain, phone, or
  website.
- **Founders** (Daniel and JB) decide by consensus while the project is small. Substantive
  changes to the spec, license, or governance are proposed as pull requests, so the reasoning
  stays public.

## Money and neutrality

- The raw data stays free (CC0), forever. Paywalling it would destroy the trust that makes it
  valuable.
- Money is made on top: apps (Plinkato, PianoMeetups), services, sponsorships. This is the
  OpenStreetMap model: the map is free, an industry is built on it.
- When there's a concrete reason (grants, partners, money), incorporate a non-profit foundation
  and move the GitHub org and domains into it. Until then, a co-owned GitHub org with the
  founders as equal owners is enough to be real and neutral.
