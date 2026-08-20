# OpenPianos — Contributing

Three ways data changes: edits, verifications, and imports. All three are logged; nothing is ever
permanently deleted.

## Editing (anyone with an account)

Like Wikipedia: open the piano, change what's wrong, save. Adding a new piano is creating a new
page. Every edit records who made it and when, and the full revision history is kept, so any edit
can be reverted. New accounts' edits may be flagged for ambassador review while trust is built;
that's a review queue, not a submission wall.

## Verifying

The single most valuable contribution is the simplest one: **"I'm here, the piano is still
here."** One tap on the site (or in a consuming app) updates the piano's `lastVerifiedAt` and
records who verified it and how (in person, QR scan on the piano, photo, phone call to the
venue). The map shows freshness, so a piano verified last week reads differently from one nobody
has seen since 2019.

Reporting a piano **gone** works the same way: a "no longer here" verification. An ambassador
confirms it (or the reports pile up) and the status flips to `removed`. The page stays, history
intact, and can be revived if the piano returns.

## Ambassadors

An ambassador adopts a scope: one piano, a venue, a city, a state, or a country. Within it they:

- review recent edits and fix mistakes,
- verify pianos (their verification is the strongest freshness signal),
- settle disputes (is this a duplicate? was that removal real?),
- welcome and guide new contributors in their area.

Their dashboard shows what changed in their scope, what's unverified the longest, and what's been
reported gone.

## Venue operators

Whoever runs the venue has the freshest knowledge. An operator can claim their venue, verify the
claim (official email domain, phone, or a token on the venue's website), and then maintain their
own pianos directly: hours, access rules, a removal, a new grand. Operator edits are ordinary
attributed edits; they don't erase history.

## Imports (bulk sources)

Seeding and syncing from existing datasets (pianos.pub, rail operators, community maps; see
`SOURCES.md`):

- Each imported record remembers its source and the source's id, so re-imports update the same
  piano instead of duplicating it.
- Hand-made corrections survive re-syncs: if a human merged or removed a piano, a re-import of
  the same source record respects that.
- An import never overwrites a fresher human edit.
- Only import sources whose license or explicit permission allows CC0 redistribution (see
  `LICENSE`); when in doubt, ask the source first.

## Photos and comments

The dataset stores **links, not media**: a photo is a link to where it lives, never the image
file itself. Free-text comments stay in the apps that collected them; what OpenPianos stores is
the useful fact ("out of tune", "ask reception for the key") plus a link back. This keeps the
dataset small, portable, and free of copyright and privacy problems, which matters because CC0
publication is irrevocable.

## Changing the spec

The model, governance, and license live in this repo. Propose changes as pull requests so the
reasoning stays public and versioned.
