# bookshelf — Product

**Status:** `draft`
**Author:** Eric Caskey
**Created:** 2026-04-20

> This is a **fictional** domain-package example. `bookshelf` is a
> small greenfield personal reading tracker, designed to illustrate the
> `product / domain / contracts / delivery` shape defined in ADR-001.
> No such product exists. The shape is real; the content is pedagogical.

---

## Why

People who read a lot of books lose track. They forget what they've
read, they re-buy books they already own, they abandon books midway and
forget why, they don't notice patterns in their own taste until a
friend points one out.

`bookshelf` is a single-user CLI + local web UI for tracking what
you're reading, what you've read, and what you thought about it. Local
files, no cloud, no social features.

## Who

One user: the person who installed it. No multi-tenancy, no accounts.
`~/.bookshelf/` holds everything.

## What (product-level capabilities)

1. **Add a book.** ISBN, title, author, format (paper / e-book /
   audio). Optional: acquired date, cover image URL.
2. **Start a reading session.** Pick a book, record a start timestamp,
   optional location and mood.
3. **End a reading session.** Record end timestamp, pages read (or
   percent for audio/e-book), optional note.
4. **Mark a book.** `reading`, `finished`, `abandoned`, `reference`
   (dipped into, not linearly read).
5. **Review a book.** Rating 1–5, a freeform markdown review, optional
   tags.
6. **Query.** "What did I read in 2025?" "Which authors have I read more
   than three books from?" "Average session length by time of day?"

## Use cases (narrative)

- **The weekly review.** On Sunday mornings, I run `bookshelf this-week`
  and see everything I read this week with time spent and notes.
- **The abandoned book check.** Before starting a new book, I run
  `bookshelf abandoned` to see which books I put down and why —
  sometimes it's worth picking one back up.
- **The pattern.** I run `bookshelf patterns` after a year and it tells
  me I read mostly in the morning, mostly nonfiction, and that my
  ratings trend higher for shorter books.

## Boundaries (what this product deliberately is NOT)

- **Not a Goodreads replacement.** No social graph, no followers, no
  public reviews. One reader, one machine.
- **Not a library catalog.** Books you don't own are fine to track;
  physical shelf location is not modeled.
- **Not a recommender.** No "books like this" feature. The user knows
  what they want next.
- **Not a cloud service.** Local SQLite file plus a directory of
  markdown review files.

## Success criteria

- One user (the author) uses it daily for 90 days without losing data.
- Data is plain-text or SQLite — always exportable by hand.
- Starting a session takes one command and under 5 seconds.
- A year of data (~500 sessions, ~60 books) queries in under 200ms.

## Open product questions

- Do we want a mobile companion for logging sessions? Leaning no — the
  CLI works from a phone over SSH.
- Audiobook chapters vs. minutes: model both or pick one? Decided in
  `domain.md §ReadingSession`.
