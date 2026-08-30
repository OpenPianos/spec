# OpenPianos

**An open, Wikipedia-style database of the world's public pianos.**

## In one line

> One shared, open dataset of public pianos that anyone can edit and trusted ambassadors keep
> honest, so every piano map and app can stop maintaining its own stale copy.

## The problem

Lots of people have mapped public pianos. Almost every map is now abandoned, because they all hit
the same wall: piano data goes stale, and stale piano data is worthless. A piano listed in 2019 may
be long gone. And because every project keeps its own copy, the same piano is entered five times,
deleted in one place and alive in another. There is no shared record.

## The model

OpenPianos works like Wikipedia:

1. **Anyone can edit.** Add a piano, fix its location, update its hours, report it gone. No
   gatekeeping to contribute. Every edit is recorded with who made it and when, and the full
   history of every piano is kept forever, so any mistake or vandalism is one click to undo.

2. **Trusted people keep it honest.** **Ambassadors** are vetted contributors — trust follows
   the person, not a polygon: an ambassador can verify, photograph, and moderate anywhere,
   while their home city stays their patrol responsibility and identity. A piano confirmed by
   an ambassador wears a **verified badge**; the record shows who vouched and when. **Curators**
   tend the dataset itself: imported leads become pianos only through a curator's decision.

3. **Sources feed it, apps build on it.** External sources (pianos.pub, rail operators,
   YouTube, airport lists — see [`SOURCES.md`](SOURCES.md)) arrive as *leads*, are curated into
   records, and keep sending signals about them afterwards. The data is free (CC0) for anyone
   to use; consuming apps like Plinkato and PianoMeetups read from OpenPianos and write back
   what their users learn. The social layer lives in the apps; OpenPianos stays the neutral
   factual core underneath.

## The product, as it stands (Aug 2026, pre-launch)

**openpianos.net** — live map with per-piano pages (local-language + English descriptions,
photos with faces blurred, hero videos, full history), one-tap "it's here" confirmations, a
freshness score on every record, QR tags to hang on pianos, a curator inbox where AI drafts and
humans decide, and open data at `/pianos.geojson`. Launch checklist is tracked in the site repo.

## What a piano record looks like

Kept deliberately small: where it is, what it is, how you get to play it, and how fresh that
knowledge is. The field list lives in [`SCHEMA.md`](SCHEMA.md).

## Read next

- [`SCHEMA.md`](SCHEMA.md) — the data model, in plain terms
- [`rfcs/0001-data-handling.md`](rfcs/0001-data-handling.md) — the normative data-handling rules
  (pipeline, mirroring, licensing, the never-do list)
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — governance, roles, how edits and verification work,
  and the decision log
- [`SOURCES.md`](SOURCES.md) — the source registry: what feeds us, under what license
- [`LICENSE`](LICENSE) — CC0, plus notes on third-party sources

## Origins & credits

OpenPianos began with the *OpenPianos — Product & Technical Brief (v1)* by **Daniel Seixas**
(PianoMeetups) — archived as [`BRIEF-v1.md`](BRIEF-v1.md) with a mapping to what was built. An
earlier revision of this spec explored an event-sourced observation model; the project moved to
the simpler wiki + ambassadors model described here (decision log in `CONTRIBUTING.md`). The git
history preserves the earlier design.

Co-founded by **Daniel Seixas** (PianoMeetups) and **JB** (Plinkato).

## License

The data is public domain under [CC0 1.0](LICENSE). Attribution is requested as a community norm,
not a legal requirement. See `LICENSE` for the note on OpenStreetMap and other third-party sources
that cannot be relicensed as CC0.
