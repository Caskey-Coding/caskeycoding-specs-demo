# bookshelf — Rollout Plan

**Status:** `draft`
**Author:** Eric Caskey (demo author)
**Created:** 2026-04-20
**Updated:** 2026-04-20

> Phased plan from skeleton to v1. Each phase has an explicit exit
> condition. Acceptance scenarios that gate each phase are in
> [`./001-acceptance-tests.md`](./001-acceptance-tests.md).

## Phases

### Phase 1 — Walking skeleton (Week 1)

- `bookshelf add` and `bookshelf start` / `bookshelf end` working
  end-to-end against SQLite.
- No web UI. No review. No query subcommands.
- Unit tests for the domain invariants in
  [`../domain/001-model.md`](../domain/001-model.md).
- One integration test: add a book, start a session, end it, verify
  the session is in the DB.

**Exit condition:** the author uses `bookshelf` to track real reading
for 7 consecutive days without data loss.

### Phase 2 — Review and query (Week 2)

- `bookshelf review` with `$EDITOR` integration, writing review
  files per [`../domain/002-data-contracts.md §Review file format`](../domain/002-data-contracts.md).
- `bookshelf status` for the full state machine.
- `bookshelf query this-week`, `year`, `abandoned`, `patterns`.
- Raw SQL read-only query via `bookshelf q 'SELECT ...'`.

**Exit condition:** a one-month retrospective ("what did I read in
March") runs in < 200ms on real data.

### Phase 3 — Web UI (Weeks 3–4)

- Local HTTP API per [`../contracts/001-api.md`](../contracts/001-api.md).
- Minimal web UI: book list, book detail, session log, review
  viewer.
- Served from the same binary: `bookshelf web --port 8080`.
- No auth; bound to loopback.

**Exit condition:** the author logs a session from a phone browser
over SSH tunnel in < 10 seconds.

### Phase 4 — Polish and release (Week 5)

- Export and import (`bookshelf export > bookshelf.json`).
- Release notes from conventional commits.
- Homebrew formula, `go install` path.
- README with screenshots.

**Exit condition:** someone other than the author installs it and
completes a "add a book, log a session" round trip without asking
a question.

## Migrations

Schema changes go through `migrations/NNN_description.sql`. On
startup, `bookshelf` runs any pending migrations and records them in
a `schema_migrations` table. No migration tooling beyond this — the
migration story is "run them in order, forward only." Rollbacks are
out of scope; if a migration is wrong, write a new forward migration.

Soft deletes: `Book.deleted_at` timestamp. The query layer filters
it out. Hard delete is a manual CLI subcommand, separate PR, Phase
4+.

## Observability

- CLI: `--log-level` flag; default `info` to stderr.
- Web server: structured access log to stderr, one line per request.
- No telemetry. No phone-home. No error reporting service.
- Diagnostics: `bookshelf doctor` prints DB path, migration version,
  disk usage, last session timestamp, and divergences between
  markdown reviews and the projection.

## Acceptance for v1.0.0

- [ ] All four phases shipped behind a tag.
- [ ] 90 consecutive days of use by the author without data loss.
- [ ] `go install github.com/example/bookshelf/cmd/bookshelf@v1`
      works on Linux and macOS.
- [ ] README quickstart completes in under 5 minutes for a new user.
- [ ] No open `critical` or `high` severity issues.

## Risks

| Risk | Mitigation |
|------|------------|
| SQLite file corruption on crash | WAL mode; `bookshelf doctor` detects and reports |
| User edits a review markdown file into invalid frontmatter | Parser warns and ignores; CLI offers `bookshelf review --repair` |
| ISBN lookup service goes away | Lookup is optional; manual entry always works |
| Schema migration ships broken | Forward-only migrations; any break is a same-day forward fix |
| 10k+ sessions slow down queries | Indexes on `(book_id, ended_at)`; `bookshelf doctor` reports table sizes |

## Open questions

- The exact statistics surfaced by `bookshelf q patterns` are subject
  to taste and should be revisited after the first year of real use.
- Whether to provide a built-in importer for Goodreads exports.
  Currently `out of scope` per
  [`../product/001-requirements.md`](../product/001-requirements.md);
  may revisit if many users ask.
- Whether the SQLite database benefits from a periodic
  `VACUUM`. Probably yes, monthly, in `bookshelf doctor`.

## Rollout

No rollout. It's a single-user local-file tool. Tag v1.0.0, publish
binaries via goreleaser, done. Updates are user-initiated
(`go install ...@latest`).

## What could change this plan

- A real user reports flow that doesn't fit the current domain model.
  That's an ADR moment and may push out a phase.
- Performance acceptance fails on a 5k-session corpus. Push out
  Phase 4 to add the missing index and re-test.
- A second user requests a multi-user mode. That's not a phase
  shift; it's a new package (a second domain context), tracked
  separately.
