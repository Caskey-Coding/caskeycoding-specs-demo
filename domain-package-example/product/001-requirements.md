# bookshelf — Product Requirements

**Status:** `draft`
**Author:** Eric Caskey (demo author)
**Created:** 2026-04-20
**Updated:** 2026-04-20

## Why this product exists

People who read a lot of books lose track. They forget what they have
read, they re-buy books they already own, they abandon books midway and
forget why, they don't notice patterns in their own taste until a
friend points one out.

`bookshelf` is a single-user CLI plus local web UI for tracking what
you are reading, what you have read, and what you thought about it.
Local files. No cloud. No social features.

## Users

One. The person who installed it. No multi-tenancy, no accounts.
`~/.bookshelf/` holds everything.

### Personas

- **The marathon reader.** 40+ books a year. Wants to look back and
  see patterns. Cares about completion stats and abandoned-book
  reasons.
- **The reference reader.** Many books open simultaneously, dipped
  into rather than read linearly. Cares about being able to find a
  passage they remember reading.
- **The session-tracker.** Logs every reading session by time and
  location. Cares about session-level stats and habit reinforcement.

The domain model (see [`../domain/001-model.md`](../domain/001-model.md))
is designed so all three personas share one schema — the difference is
which queries they run.

## Goals

1. Make adding a book a one-command, sub-5-second operation.
2. Make starting and ending a reading session friction-free enough
   that a user actually does it every session.
3. Preserve the complete event history so retroactive queries
   ("what did I read in 2025?") are always answerable.
4. Keep the data file plain enough that it survives the tool — even
   if `bookshelf` itself is unmaintained, the SQLite + markdown
   contents remain readable.
5. Make the daily read fast on a year of data (~500 sessions, ~60
   books): all queries < 200ms.

## Non-goals

- **Not a Goodreads replacement.** No social graph, no followers, no
  public reviews. One reader, one machine.
- **Not a library catalog.** Books you do not own are fine to track;
  physical shelf location is not modeled.
- **Not a recommender.** No "books like this" feature. The user
  knows what they want next.
- **Not a cloud service.** Local SQLite + a directory of markdown
  reviews. Sync is the user's responsibility.
- **Not a reading-rate optimizer.** No streaks, no gamification, no
  daily-goal nags.

## Success criteria (measurable)

- One user (the author) uses it daily for 90 days without losing
  data.
- Data is plain-text or SQLite — always exportable by hand.
- Starting a session takes one command and under 5 seconds.
- A year of data queries in under 200ms.
- A new user completes the README quickstart in under 5 minutes.

## Status lifecycle for the product

| Status | Meaning |
|--------|---------|
| `draft` | This file is being written; expect churn |
| `ready` | Reviewed; safe for an implementer to start |
| `in-progress` | Implementation underway; deviations noted in delivery/002 |
| `completed` | Shipped; Implementation Reality reflects current code |
| `deprecated` | Replaced by another spec (linked from this one) |

Current status: `draft`. See [`../delivery/002-rollout-plan.md`](../delivery/002-rollout-plan.md)
for the path to `ready`.

## References

- [Use cases](./002-use-cases.md) — narrative flows for the three
  personas
- [Domain model](../domain/001-model.md) — the entities and
  invariants that make the goals above realizable
- [Acceptance tests](../delivery/001-acceptance-tests.md) — how each
  goal is verified
