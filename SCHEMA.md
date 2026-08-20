# OpenPianos — Data model

Plain-language draft of the simplified model (see `DECISIONS.md` ADR-0005; the earlier
event-sourced design is in git history). Field lists are a starting point for the first website,
not a contract; expect this to move while we build.

## Piano

The atom of the dataset. One record per physical piano.

| Field | Meaning |
|---|---|
| `id` | Permanent opaque id. Never reused; redirects if merged. |
| `name` | Display name ("St Pancras station piano"). |
| `lat`, `lon` | Location. |
| `address`, `city`, `region`, `country` | Human-readable place. |
| `venue` | Optional link to a Venue (see below). |
| `access` | `public` (walk up and play) · `bookable` (studio/practice room) · `ask` (venue piano, ask staff). |
| `hours` | When it's playable, free text for now. |
| `status` | `active` · `temporary` (with `activeFrom`/`activeUntil`) · `needs_verification` · `removed`. |
| `instrument` | Optional: type/brand ("upright, Yamaha"). |
| `notes` | Short practical facts ("out of tune", "key at reception"). |
| `links` | External links: photos, source pages, videos. Links only, never media files. |
| `lastVerifiedAt` / `lastVerifiedBy` | The freshness signal: when someone last confirmed it, and who. |
| `sources` | Where the record came from (e.g. `pianos.pub:84d11559`), so imports update instead of duplicate. |

## Revision history

Every change to a piano is a revision: who, when, what changed, optional note. Records are edited
in place (wiki-style), but no information is lost, because the full revision chain is kept and any
revision can be reverted. `removed` pianos keep their page and history.

## Verification

A small record attached to a piano: who verified, when, the verdict (`present` / `gone`), and how
(`in_person`, `qr_scan`, `photo`, `phone`, `operator`). The newest verification drives
`lastVerifiedAt` and the freshness shown on maps. Consuming apps write these too: a Plinkato
"plink" (a QR-verified visit) arrives as a `present` verification.

## Account

| Field | Meaning |
|---|---|
| `id`, `name`, `email` | The basics. |
| `role` | `contributor` (default) · `ambassador` · `admin`. |
| `ambassadorScopes` | For ambassadors: what they steward — a piano id, a venue id, a city, a state, or a country. One account can hold several scopes. |

Consuming apps (Plinkato, PianoMeetups) get API credentials and pass through which of their users
made each edit, so attribution stays about people, not apps.

## Venue (optional grouping)

For places that host several pianos or bookable rooms: a station, a library, a piano studio.
Carries the venue-level facts (name, location, website, opening hours, operator contact) so its
pianos don't repeat them. A venue operator who verifies a claim maintains this record and its
pianos.

## Exports & API

- Open exports: GeoJSON, CSV, JSON, refreshed on every change or on a schedule.
- A read API for maps and apps, and a write API for edits and verifications from consuming apps.
- A change feed (what changed since X) so consumers can sync instead of re-downloading.

## Deliberately not modeled (yet)

- Confidence scores and trust tiers: replaced by visible attribution + `lastVerifiedAt` + who
  verified. If spam ever demands more, we add it then.
- Stored media: links only.
- Recurring seasonal pianos: a returning festival piano is re-activated (or re-added) per season.
