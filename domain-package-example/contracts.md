# bookshelf — Contracts

**Status:** `draft`
**Author:** Eric Caskey
**Created:** 2026-04-20

> Contracts are the stable interfaces the domain exposes. Read
> `domain.md` first. This document describes the CLI, the local HTTP
> API (for the web UI), and the file-on-disk contracts — what the
> outside world is allowed to depend on.

---

## CLI contract

### `bookshelf add`

```
bookshelf add --isbn 978-0-13-468599-1
bookshelf add --title "A Pattern Language" \
              --author "Christopher Alexander" \
              --format paper
```

At least one of `--isbn` or (`--title` + `--author`) is required.
`--format` defaults to `paper`. Exit `0` on add, `2` on validation
error, `3` on storage error.

### `bookshelf start`

```
bookshelf start <book-id-or-title-substring>
        [--location "train"]
        [--mood focused]
```

Opens a session. Errors if one is already open for that book.

### `bookshelf end`

```
bookshelf end <book-id-or-title-substring>
      --pages 42            # or --minutes 25 for audiobook
      [--note "chapter 3"]
```

Closes the open session. Records final progress.

### `bookshelf status`

```
bookshelf status <book-id-or-title-substring> \
         --to finished|abandoned|reading|reference \
         [--reason "lost interest"]
```

### `bookshelf review`

```
bookshelf review <book-id> --rating 4 \
         [--tags food,memoir]
# opens $EDITOR with a template; saves on write-quit
```

### `bookshelf query` / `bookshelf q`

```
bookshelf q this-week
bookshelf q year 2025
bookshelf q abandoned
bookshelf q patterns
bookshelf q 'SELECT ...'   # raw SQL, read-only
```

## Local HTTP API

Used by the web UI (`bookshelf web`). Binds to `127.0.0.1` only. No
auth — single-user, single-machine assumption.

### Endpoints

```
GET  /api/books                          → []BookSummary
GET  /api/books/{id}                     → BookDetail
POST /api/books                          → BookDetail
PATCH /api/books/{id}                    → BookDetail

POST /api/books/{id}/sessions            → Session            (start)
PATCH /api/sessions/{id}                 → Session            (end)
GET  /api/sessions?book_id=...           → []Session

PUT  /api/books/{id}/review              → Review
GET  /api/books/{id}/review              → Review

GET  /api/query/this-week                → []Session
GET  /api/query/patterns                 → PatternsReport
```

All JSON. Timestamps are RFC 3339 UTC. Versioning: URL prefix is `/api`
without a version — single consumer, breaking changes are fine before
1.0. Post-1.0, move to `/api/v1`.

### Shapes (excerpt)

```json
BookSummary {
  "id": "978-0-13-468599-1",
  "title": "A Pattern Language",
  "authors": ["Christopher Alexander"],
  "format": "paper",
  "status": "reading",
  "last_session_ended_at": "2026-04-19T08:14:00Z"
}

Session {
  "id": "01HV9...",
  "book_id": "978-0-13-468599-1",
  "started_at": "2026-04-20T07:30:00Z",
  "ended_at": "2026-04-20T08:05:00Z",
  "progress": { "pages_read": 12, "page_count": 1171 },
  "mood": "focused",
  "note": "intro"
}
```

Errors follow RFC 7807 Problem+JSON.

## File contracts (on-disk)

Everything lives under `~/.bookshelf/` (override with `$BOOKSHELF_HOME`).

```
~/.bookshelf/
├── data.sqlite                  ← the event store + materialized views
├── reviews/
│   └── 978-0-13-468599-1.md     ← one markdown file per review
└── config.yaml                  ← editor preference, HTTP port, etc.
```

### Review file format

```markdown
---
book_id: 978-0-13-468599-1
rating: 5
written_at: 2026-04-19T22:00:00Z
tags: [architecture, design]
---

# A Pattern Language

Freeform review body in markdown.
```

Frontmatter fields are authoritative. A review file without valid
frontmatter is ignored with a warning on startup.

## Stability promises

| Surface | Stability |
|---------|-----------|
| CLI verb names (`add`, `start`, `end`, `status`, `review`, `query`) | Stable post-1.0 |
| CLI flag names | Stable post-1.0 |
| Local HTTP API JSON shapes | Stable post-1.0, URL-versioned |
| SQLite schema | Internal; migrations handle changes |
| Review markdown file format | Stable — users may hand-edit these |
| `config.yaml` keys | Stable post-1.0 |

Breaking changes to "Stable" surfaces require a new ADR and a major
version bump.
