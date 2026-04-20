# bookshelf — Domain Model

**Status:** `draft`
**Author:** Eric Caskey (demo author)
**Created:** 2026-04-20
**Updated:** 2026-04-20

> The domain model is the center of gravity of a domain package. Read
> [`../product/001-requirements.md`](../product/001-requirements.md)
> first for the why. This document describes the entities, value
> objects, invariants, and state transitions that the rest of the
> system must respect.

## Bounded context

One. There is a single bounded context: "the user's personal reading
life." Everything in the system is owned by one user. There is no
multi-tenant aggregate boundary, no inter-context translation layer.
This is the smallest possible domain shape and one of the things
that makes `bookshelf` a good demo of the archetype.

## Entities

### Book

The thing being read. Identity is the ISBN-13 when one exists;
otherwise a generated UUID.

```
Book {
  id:        BookID         // ISBN-13 or UUID
  title:     string         // required, max 500
  authors:   []string       // 1..N, each max 200
  format:    Format         // paper | ebook | audiobook
  acquired:  Date?          // when added to the shelf
  cover_url: URL?           // optional image
  status:    BookStatus     // reading | finished | abandoned | reference
  tags:      []Tag          // 0..N freeform labels
}
```

**Invariants:**

- `status=finished` requires at least one completed `ReadingSession`.
- `status=abandoned` requires a non-empty `reason` string on the
  accompanying `StatusChanged` event.
- Once `status=finished`, no new `ReadingSession` can be opened
  against this book. To resume reading, explicitly transition to
  `status=reading` (which records a `StatusChanged` event).

### ReadingSession

One contiguous reading effort. Opens with `start`, closes with `end`.

```
ReadingSession {
  id:         SessionID
  book_id:    BookID
  started_at: Instant
  ended_at:   Instant?      // nil while open
  progress:   Progress      // see value objects
  location:   string?       // freeform: "train", "backyard"
  mood:       Mood?
  note:       string?       // markdown
}
```

**Invariants:**

- At most one `ReadingSession` per `book_id` may have `ended_at=nil`
  at any time.
- `ended_at > started_at` when set.
- `progress` at session end must be monotonically non-decreasing vs.
  the previous closed session for the same book. Re-reading a book
  is a status transition and a new chain of sessions, not a
  monotonicity violation.

### Review

```
Review {
  book_id:    BookID         // 1:1 with Book
  rating:     Rating         // 1..5 integer
  body:       string         // markdown
  written_at: Instant
  tags:       []Tag
}
```

**Invariants:**

- A review can only exist for a book whose current `status` is
  `finished` or `abandoned`.

## Value objects

### Format

Enum: `paper`, `ebook`, `audiobook`.

### Progress

Discriminated by `Book.format`:

- `paper` and `ebook`: `pages_read: int`, `page_count: int?`.
  Derived `percent = pages_read / page_count` when `page_count` is
  known.
- `audiobook`: `minutes_listened: int`, `total_minutes: int?`.

### Mood

Enum: `focused`, `drifty`, `tired`, `energized`, `neutral`.

### Rating

Integer 1–5. No half-stars. No decimals.

### Tag

Lowercase kebab-case string, max 40 chars. No spaces. No emoji.

## Aggregates

`Book` is an aggregate root. `ReadingSession` and `Review` live under
it. Deleting a `Book` deletes all its sessions and its review. (Soft
delete only; see [`../delivery/002-rollout-plan.md §Migrations`](../delivery/002-rollout-plan.md).)

## Events (state transitions)

The store is append-only on events, with materialized current-state
views used for queries.

| Event | Preconditions | Effect |
|-------|---------------|--------|
| `BookAdded` | Book with this ID does not exist | Creates Book with `status=reading` |
| `StatusChanged` | Book exists | Updates `status`; records reason if `abandoned` |
| `SessionStarted` | No open session on this book | Creates open ReadingSession |
| `SessionEnded` | Open session exists | Closes it with final `ended_at` and `progress` |
| `ReviewWritten` | Book status is `finished` or `abandoned`, no review exists | Creates Review |
| `ReviewEdited` | Review exists | Replaces `body` and `rating`; keeps `written_at` |
| `BookDeleted` | Book exists | Soft-deletes (tombstone); cascading hide of sessions/review |

The full on-disk encoding of these events is in
[`./002-data-contracts.md §Event row schema`](./002-data-contracts.md).

## State machine: Book.status

```
               +-----------+
               |  reading  |
               +-----+-----+
                /    |    \
               /     |     \
              v      v      v
      +----------+ +-----+ +-----------+
      | abandoned| | ref | | finished  |
      +----+-----+ +--+--+ +----+------+
           ^          ^         ^
           |          |         |
           +----------+---------+
              all transitions are recorded
              as StatusChanged events; cycles
              are allowed (re-read → reading)
```

## Rules that are NOT in the domain

These are application concerns. They live elsewhere:

- Command-line syntax for adding a book → [`../contracts/001-api.md`](../contracts/001-api.md)
- SQLite schema → [`./002-data-contracts.md`](./002-data-contracts.md)
- File layout of markdown reviews on disk → [`./002-data-contracts.md`](./002-data-contracts.md)
- ISBN lookup against external services → [`../contracts/001-api.md §CLI`](../contracts/001-api.md)

## Glossary

- **Open session** — a `ReadingSession` with `ended_at=nil`.
- **Progress monotonicity** — you can only move forward in a book
  between sessions in the same status streak. If you change status
  to `abandoned` and back to `reading`, that opens a new streak;
  the prior chain still counts but no longer constrains.
- **Reference book** — dipped into, not read linearly. Sessions
  still welcome; progress tracking is loose.
