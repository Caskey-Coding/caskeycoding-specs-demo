# bookshelf — Acceptance Tests

**Status:** `draft`
**Author:** Eric Caskey (demo author)
**Created:** 2026-04-20
**Updated:** 2026-04-20

> Acceptance criteria are split from rollout planning so the test
> definitions stay stable as the rollout plan shifts. Every checkbox
> here corresponds to one or more goals in
> [`../product/001-requirements.md §Success criteria`](../product/001-requirements.md).

## Coverage targets

| Layer | Tool | Target |
|-------|------|--------|
| Domain invariants | Go unit tests | 100% of rules in [`../domain/001-model.md §Invariants`](../domain/001-model.md) |
| Event store | Go integration tests against real SQLite | 85% |
| CLI | Cobra + golden-file output tests | All happy paths + top 10 error paths |
| HTTP API | `httptest` + contract tests per [`../contracts/001-api.md`](../contracts/001-api.md) | All endpoints, all documented error shapes |
| Web UI | Playwright, 3 critical paths | Add a book; log a session; write a review |

## Acceptance scenarios (Gherkin-ish)

### Add a book and start a session

```
Given a fresh ~/.bookshelf/
When I run `bookshelf add --isbn 978-0-13-468599-1`
Then exit code is 0
 And the BookAdded event is in the store
 And `bookshelf q this-week` lists the book

When I run `bookshelf start 978-0-13-468599-1`
Then exit code is 0
 And SessionStarted event is in the store
 And the session has ended_at = NULL
```

### Cannot open two sessions on the same book

```
Given a book with one open session
When I run `bookshelf start <same book>`
Then exit code is 2
 And stderr contains "session already open"
 And no new event is written
```

### Re-reading: status → abandoned → reading

```
Given a book with status=abandoned and a recorded reason
When I run `bookshelf status <book> --to reading`
Then exit code is 0
 And StatusChanged event is in the store with to=reading
 And the existing sessions are still attached to the book
 And the next `bookshelf start` succeeds
```

### Review only after finished or abandoned

```
Given a book with status=reading
When I run `bookshelf review <book> --rating 4`
Then exit code is 2
 And stderr contains "review requires status finished or abandoned"
```

### Web UI parity with CLI

```
Given the web server running on 127.0.0.1:8080
When I POST /api/books with a valid BookCreate body
Then the response is 200 and matches the CLI's `bookshelf add` postconditions
```

More scenarios live alongside the implementation; this list is the
minimum that must pass before tagging.

## Performance acceptance

- A year of data (~500 sessions, ~60 books) returns `bookshelf q
  this-week` in under 200ms (p95) on the developer baseline machine.
- 10x scale (5000 sessions) returns the same query in under 2s.
  Beyond that, indexes need a revisit; tracked as an open question.
- Cold-start of `bookshelf` CLI (any subcommand) under 100ms,
  excluding I/O on first DB-open.

## Spec lint

For this demo we deliberately do **not** ship frontmatter validators
(see ADR-006 in `caskeycoding-specs`). In the live repo, every spec
file passes:

- A frontmatter shape check (`Status`, `Author`, `Created`, `Updated`).
- A markdown lint pass (rules in `.markdownlint.yml`).
- An NDA scan (allow-list of brand names; deny-list of NDA-bound
  partners and clients).

This demo repo is human-curated, refreshed at milestones. Validation
is the writer's responsibility.

## What "green" means

A build is green for a 1.0 candidate when:

- All scenarios above pass.
- All performance budgets pass on the developer baseline machine.
- `go test ./... -race` passes.
- Playwright critical-path suite passes against a fresh `~/.bookshelf/`.
- No `TODO(blocking)` comments in `internal/`.
