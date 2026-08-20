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

2. **Ambassadors keep it trustworthy.** An ambassador adopts a scope: a single piano, a venue, a
   city, a state, or a country. Within that scope they review contributions, verify pianos are
   really there, and correct what's wrong. A piano recently checked by its ambassador is the gold
   standard of freshness; the site shows when each piano was last verified and by whom.

3. **Apps build on top, and give back.** The data is free (CC0) for anyone to use. Consuming apps
   like Plinkato and PianoMeetups read from OpenPianos instead of keeping their own piano
   databases, and they write back what their users learn: a Plinkato user playing a piano today is
   evidence it exists, and that verification flows into the shared record. The social layer
   (profiles, passports, streams, community) lives in the apps; OpenPianos stays the neutral
   factual core underneath them.

## The first product

A simple website where this happens: browse the map, sign up for an account, add a piano, edit
one, press "I verified this piano is still here." Ambassadors get a dashboard for their scope
(what changed, what needs checking, what was reported gone). Naming leans toward an
OpenPianos-branded site; publicpiano.info is available as an alternative.

## What a piano record looks like

Kept deliberately small: where it is, what it is, how you get to play it, and how fresh that
knowledge is. The current field list lives in [`SCHEMA.md`](SCHEMA.md).

## Read next

- [`SCHEMA.md`](SCHEMA.md) — the data model, in plain terms
- [`GOVERNANCE.md`](GOVERNANCE.md) — who owns it (nobody), how it's run
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — how edits, verification, and ambassadors work
- [`SOURCES.md`](SOURCES.md) — existing datasets we can seed from, with licenses
- [`DECISIONS.md`](DECISIONS.md) — the decision log, including the switch to this simpler model
- [`LICENSE`](LICENSE) — CC0, plus notes on third-party sources

## Origins & credits

OpenPianos began with the *OpenPianos — Product & Technical Brief (v1)* by **Daniel Seixas**
(PianoMeetups): the canonical-open-dataset thesis and the dataset + API + contribution pillars.
An earlier, more technical revision of this spec explored an event-sourced observation model; the
project has since deliberately moved to the simpler wiki + ambassadors model described here (see
`DECISIONS.md`, ADR-0005). The git history preserves the earlier design.

Co-founded by **Daniel Seixas** (PianoMeetups) and **JB** (Plinkato).

## License

The data is public domain under [CC0 1.0](LICENSE). Attribution is requested as a community norm,
not a legal requirement. See `LICENSE` for the note on OpenStreetMap and other third-party sources
that cannot be relicensed as CC0.
