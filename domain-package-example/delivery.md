# bookshelf — Delivery

**Status:** `draft`
**Author:** Eric Caskey
**Created:** 2026-04-20

> Delivery covers how the product gets built, tested, released, and
> evolved. Read `product.md`, `domain.md`, and `contracts.md` first —
> this document is about execution, not shape.

---

## Phases

### Phase 1 — Walking skeleton (Week 1)

- `bookshelf add` and `bookshelf start` / `bookshelf end` working
  end-to-end against SQLite.
- No web UI. No review. No query subcommands.
- Unit tests for the domain invariants in `domain.md`.
- One integration test: add a book, start a session, end it, verify
  the session is in the DB.

Exit condition: the author uses `bookshelf` to track their own reading
for 7 consecutive days without data loss.

### Phase 2 — Review and query (Week 2)

- `bookshelf review` with `$EDITOR` integration.
- `bookshelf status` for the full state machine.
- `bookshelf query this-week`, `year`, `abandoned`, `patterns`.
- Raw SQL read-only query via `bookshelf q 'SELECT ...'`.

Exit condition: a one-month retrospective ("what did I read in March")
runs in < 200ms on real data.

### Phase 3 — Web UI (Week 3–4)

- Local HTTP API per `contracts.md`.
- Minimal web UI: book list, book detail, session log, review viewer.
- Served from the same binary: `bookshelf web --port 8080`.
- No auth; bound to loopback.

Exit condition: the author can log a session from a phone browser over
SSH tunnel in < 10 seconds.

### Phase 4 — Polish and release (Week 5)

- Export and import (`bookshelf export > bookshelf.json`).
- Release notes from conventional commits.
- Homebrew formula, `go install` path.
- README with screenshots.

Exit condition: someone other than the author installs it and completes
a "add a book, log a session" round trip without asking a question.

## Testing strategy

| Layer | Tool | Coverage target |
|-------|------|-----------------|
| Domain invariants | Go unit tests | 100% of rules in `domain.md §Invariants` |
| Event store | Go integration tests against real SQLite | 85% |
| CLI | Cobra + golden-file output tests | All happy paths + top 10 error paths |
| HTTP API | `httptest` + contract tests per `contracts.md` | All endpoints, all documented error shapes |
| Web UI | Playwright, 3 critical paths | Add a book; log a session; write a review |

## Migrations

Schema changes go through `migrations/NNN_description.sql`. On startup,
`bookshelf` runs any pending migrations and records them in a
`schema_migrations` table. No migration tooling beyond this — the
migration story is "run them in order, forward only." Rollbacks are
out of scope; if a migration is wrong, write a new forward migration.

Soft deletes: `Book.deleted_at` timestamp. The query layer filters it
out. Hard delete is a manual CLI subcommand, separate PR, Phase 4+.

## Observability

- CLI: `--log-level` flag; default `info` to stderr.
- Web server: structured access log to stderr, one line per request.
- No telemetry. No phone-home. No error reporting service.
- Diagnostics: `bookshelf doctor` prints DB path, migration version,
  disk usage, last session timestamp.

## Acceptance (for v1.0.0)

- [ ] All four phases shipped behind a tag.
- [ ] 90 consecutive days of use by the author without data loss.
- [ ] `go install github.com/example/bookshelf/cmd/bookshelf@v1` works
      on Linux and macOS.
- [ ] README quickstart completes in under 5 minutes for a new user.
- [ ] No open `critical` or `high` severity issues.

## Rollout plan

No rollout. It's a single-user local-file tool. Tag v1.0.0, publish
binaries via goreleaser, done. Updates are user-initiated
(`go install ...@latest`).

## What could go wrong

| Risk | Mitigation |
|------|------------|
| SQLite file corruption on crash | WAL mode; `bookshelf doctor` detects and reports |
| User edits a review markdown file into invalid frontmatter | Parser warns and ignores; CLI offers `bookshelf review --repair` |
| ISBN lookup service goes away | Lookup is optional; manual entry always works |
| Schema migration ships broken | Migrations are forward-only and small; any break is a new forward migration, same day |
| User accumulates 10k+ sessions and queries slow down | Indexes on `(book_id, ended_at)`; SQLite handles this size trivially; monitored by `bookshelf doctor` |

## Out of scope for v1

- Sync between machines.
- Multi-user.
- Public API (beyond local HTTP on loopback).
- Mobile app.
- Import from Goodreads / StoryGraph / LibraryThing.
- Backups — user's responsibility; it's a file.
