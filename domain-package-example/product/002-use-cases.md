# bookshelf — Use Cases

**Status:** `draft`
**Author:** Eric Caskey (demo author)
**Created:** 2026-04-20
**Updated:** 2026-04-20

> This document tells the product through narrative. Every flow here
> exists to ground the abstract domain model and contracts. If a flow
> is impossible to express in the model in [`../domain/001-model.md`](../domain/001-model.md)
> or the API in [`../contracts/001-api.md`](../contracts/001-api.md),
> the spec is wrong, not the flow.

## UC-1 — Add a book on Sunday morning

**Persona:** The marathon reader.

**Trigger:** Just bought a new book.

**Flow:**

1. User opens a terminal.
2. Types `bookshelf add --isbn 978-0-374-71575-0`.
3. `bookshelf` records the ISBN, fetches title and author from the
   local cache (or prompts the user if no cache hit), creates a
   `Book` with `status=reading`, prints the new ID.

**Postconditions:**

- One new event in the store: `BookAdded`.
- Book listed in `bookshelf q this-week` under "newly added."
- No reading session yet.

**Edge cases:**

- ISBN unknown to local cache and user is offline → fall back to
  prompted manual entry (`--title`, `--author`, `--format`).
- Duplicate ISBN → surface the existing book and abort. Re-adding is
  not a state transition the model supports.

---

## UC-2 — The weekly review

**Persona:** Any.

**Trigger:** Sunday morning ritual.

**Flow:**

1. User runs `bookshelf q this-week`.
2. `bookshelf` returns: books touched this week, total session time,
   pages or minutes per book, sessions grouped by day.
3. User scans the output, optionally writes a quick markdown note
   into a session via `bookshelf session note <session-id>`.

**Postconditions:**

- No new state. Read-only view over the materialized projection.

**Why this matters:** The weekly-review query is the single biggest
reason someone keeps logging sessions. It is the moment the data pays
the user back. If this query is slow or ugly, the whole product
fails. See [`../delivery/001-acceptance-tests.md §Performance`](../delivery/001-acceptance-tests.md).

---

## UC-3 — The abandoned-book check

**Persona:** The marathon reader.

**Trigger:** About to start a new book; wants to reconsider what was
set down.

**Flow:**

1. User runs `bookshelf q abandoned`.
2. `bookshelf` lists every book whose current `status=abandoned`,
   including the `reason` recorded at the `StatusChanged` event.
3. User picks one to resume; runs `bookshelf status <id> --to
   reading`.
4. `bookshelf` records a `StatusChanged` event back to `reading`.
   Future sessions on this book are allowed again.

**Postconditions:**

- One `StatusChanged` event.
- Progress accumulator does NOT reset — the historical sessions are
  still attached to the book. Session monotonicity (see
  [`../domain/001-model.md §Invariants`](../domain/001-model.md)) is
  evaluated against the most recent closed session, not against the
  status transition.

---

## UC-4 — The pattern

**Persona:** The marathon reader, after a year of use.

**Trigger:** Curious whether reading habits have shifted.

**Flow:**

1. User runs `bookshelf q patterns`.
2. `bookshelf` returns derived statistics: average session length,
   distribution by time-of-day mood, ratings vs. book length
   correlation, most-read author by session-time.
3. User reads, smiles, learns something.

**Postconditions:**

- No state change.

**Note:** The `q patterns` view is the most opinionated query in the
product. The exact statistics it surfaces are subject to taste and
should be revisited after the first year of real use. Open question
tracked in [`../delivery/002-rollout-plan.md §Open questions`](../delivery/002-rollout-plan.md).

---

## UC-5 — Session from a phone

**Persona:** Session-tracker.

**Trigger:** Sat down on a train with a book; only the phone is
available.

**Flow:**

1. User opens an SSH tunnel to the home machine and visits
   `http://127.0.0.1:8080/` in the phone browser.
2. Picks the book from a list, taps "start session."
3. The HTTP API records `SessionStarted`.
4. Forty minutes later, taps "end session," enters pages-read.
5. `SessionEnded` recorded.

**Postconditions:**

- Two events in the store. Same as if the CLI had been used directly.

**Why this matters:** The web UI is not a separate product surface.
It is a thin renderer over the same domain operations the CLI uses.
If the CLI verb works, the web button works. If it doesn't, the
domain model is the bug, not the UI.

---

## What is NOT in these use cases

- Multi-user flows (out of scope per [`./001-requirements.md §Non-goals`](./001-requirements.md)).
- Sync between machines.
- Importing from another tool.
- Anything social.

If a real user lands a flow not covered here, write a UC-N before
implementing it. New flows should reuse the existing domain events;
if they cannot, that is an ADR moment.
