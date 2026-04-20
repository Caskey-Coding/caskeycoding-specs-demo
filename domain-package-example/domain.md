# bookshelf — Domain Model

**Status:** `draft`
**Author:** Eric Caskey
**Created:** 2026-04-20

> The domain model is the center of gravity of a domain package. Read
> `product.md` first for the why. This document describes the entities,
> value objects, invariants, and state transitions that the rest of the
> system must respect.

---

## Entities

### Book

The thing being read. Identity is the ISBN-13 when one exists; otherwise
a generated UUID.

```
Book {
  id:           BookID             // ISBN-13 or UUID
  title:        string             // required, max 500
  authors:      []string           // 1..N, each max 200
  format:       Format             // paper | ebook | audiobook
  acquired:     Date?              // when added to the shelf
  cover_url:    URL?               // optional image
  status:       BookStatus         // reading | finished | abandoned | reference
  tags:         []Tag              // 0..N freeform labels
}
```

**Invariants:**

- `status=finished` requires at least one completed `ReadingSession`.
- `status=abandoned` requires a non-empty `reason` string on the
  accompanying `StatusChange` event.
- Once `status=finished`, no new `ReadingSession` can be opened against
  this book. To resume reading, explicitly transition to
  `status=reading` (which records a StatusChange).

### ReadingSession

One contiguous reading effort. Opens with `start`, closes with `end`.

```
ReadingSession {
  id:             SessionID
  book_id:        BookID
  started_at:     Instant
  ended_at:       Instant?         // nil while open
  progress:       Progress         // see value objects
  location:       string?          // freeform: "train", "backyard"
  mood:           Mood?            // value object
  note:           string?          // markdown
}
```

**Invariants:**

- At most one `ReadingSession` per `book_id` may have `ended_at=nil` at
  any time.
- `ended_at > started_at` when set.
- `progress` at session end must be monotonically non-decreasing vs. the
  previous closed session for the same book.

### Review

```
Review {
  book_id:      BookID             // 1:1 with Book
  rating:       Rating             // 1..5 integer
  body:         string             // markdown
  written_at:   Instant
  tags:         []Tag
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

- `paper` and `ebook`: `pages_read: int`, `page_count: int?`. Derived
  `percent = pages_read / page_count` when `page_count` is known.
- `audiobook`: `minutes_listened: int`, `total_minutes: int?`.

### Mood

Enum: `focused`, `drifty`, `tired`, `energized`, `neutral`.

### Rating

Integer 1–5. No half-stars. No decimals. No "4.5 but I wanted to round
up."

### Tag

Lowercase kebab-case string, max 40 chars. No spaces. No emoji.

## Aggregates

`Book` is an aggregate root. `ReadingSession` and `Review` live under
it. Deleting a `Book` deletes all its sessions and its review. (Soft
delete only; see `delivery.md §Migrations`.)

## Events (state transitions)

The store is append-only on events, with a materialized current-state
view.

| Event | Preconditions | Effect |
|-------|---------------|--------|
| `BookAdded` | Book with this ID does not exist | Creates Book with `status=reading` (default) |
| `StatusChanged` | Book exists | Updates `status`; records reason if `abandoned` |
| `SessionStarted` | No open session on this book | Creates open ReadingSession |
| `SessionEnded` | Open session exists | Closes it with final `ended_at` and `progress` |
| `ReviewWritten` | Book status is `finished` or `abandoned`, no review exists | Creates Review |
| `ReviewEdited` | Review exists | Replaces `body` and `rating`; keeps `written_at` |
| `BookDeleted` | Book exists | Soft-deletes (tombstone); cascading hide of sessions/review |

## Rules that are NOT in the domain

These are application concerns, not domain rules. They live in
`contracts.md` or `delivery.md`:

- Command-line syntax for adding a book.
- SQLite schema.
- File layout of markdown reviews on disk.
- ISBN lookup against external services.

## Glossary

- **Open session** — a ReadingSession with `ended_at=nil`.
- **Progress monotonicity** — you can only move forward in a book
  between sessions. If you re-read, that's a new Book (or a
  `StatusChanged` back to `reading` which resets progress tracking).
- **Reference book** — dipped into, not read linearly. Sessions still
  welcome; progress tracking is loose.
