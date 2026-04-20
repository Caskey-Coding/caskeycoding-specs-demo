# bookshelf — Data Contracts

**Status:** `draft`
**Author:** Eric Caskey (demo author)
**Created:** 2026-04-20
**Updated:** 2026-04-20

> Concrete on-disk encoding of the domain model in
> [`./001-model.md`](./001-model.md). Read that first.

## Storage location

Everything lives under `~/.bookshelf/` (override with
`$BOOKSHELF_HOME`):

```
~/.bookshelf/
├── data.sqlite           ← event store + materialized views
├── reviews/
│   └── 978-0-13-468599-1.md
│   └── ...
└── config.yaml           ← editor preference, HTTP port, etc.
```

The SQLite file is the source of truth for events. The markdown
review files are the source of truth for review _bodies_; the SQLite
`reviews` projection mirrors metadata for query speed but the
markdown file is canonical. On a divergence, the markdown wins; the
startup `bookshelf doctor` re-syncs the projection from disk.

## SQLite schema

### `events` (append-only)

```sql
CREATE TABLE events (
    id          TEXT    PRIMARY KEY,         -- ULID
    occurred_at TEXT    NOT NULL,            -- RFC 3339 UTC
    book_id     TEXT    NOT NULL,
    kind        TEXT    NOT NULL,            -- BookAdded, SessionStarted, ...
    payload     TEXT    NOT NULL             -- JSON, kind-discriminated shape
);

CREATE INDEX events_book_id_idx          ON events (book_id, occurred_at);
CREATE INDEX events_kind_occurred_at_idx ON events (kind, occurred_at);
```

No updates. No deletes. The only writes are inserts.
`BookDeleted` is just another event — the row stays.

### Materialized views (rebuildable)

```sql
CREATE TABLE books (
    id        TEXT PRIMARY KEY,
    title     TEXT NOT NULL,
    authors   TEXT NOT NULL,    -- JSON array
    format    TEXT NOT NULL,    -- paper | ebook | audiobook
    status    TEXT NOT NULL,    -- reading | finished | abandoned | reference
    acquired  TEXT,             -- ISO date
    cover_url TEXT,
    tags      TEXT NOT NULL,    -- JSON array
    deleted_at TEXT             -- soft-delete tombstone (NULL = live)
);

CREATE TABLE sessions (
    id          TEXT PRIMARY KEY,
    book_id     TEXT NOT NULL,
    started_at  TEXT NOT NULL,
    ended_at    TEXT,            -- NULL = open
    progress    TEXT NOT NULL,   -- JSON, format-discriminated
    location    TEXT,
    mood        TEXT,
    note        TEXT,
    FOREIGN KEY (book_id) REFERENCES books (id)
);

CREATE INDEX sessions_book_id_ended_at_idx ON sessions (book_id, ended_at);

CREATE TABLE reviews (
    book_id     TEXT PRIMARY KEY,
    rating      INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
    written_at  TEXT NOT NULL,
    tags        TEXT NOT NULL,   -- JSON array
    FOREIGN KEY (book_id) REFERENCES books (id)
);
```

The view tables are 100% derivable from `events`. `bookshelf doctor
--rebuild-views` truncates and replays. This is the safety net when
a schema migration adds a new column to a view: rebuild rather than
backfill.

## Event row schema

`events.payload` is a JSON document whose shape is determined by
`events.kind`. The on-disk shapes match the domain model field names
1:1 (with one exception, noted below).

### `BookAdded`

```json
{
  "title": "A Pattern Language",
  "authors": ["Christopher Alexander"],
  "format": "paper",
  "acquired": "2026-04-19",
  "cover_url": null,
  "tags": ["architecture", "design"]
}
```

### `StatusChanged`

```json
{
  "to": "abandoned",
  "reason": "lost interest after chapter 4"
}
```

`reason` is required when `to=abandoned`, optional otherwise. Validated
by the writer; enforced again by `bookshelf doctor`.

### `SessionStarted` / `SessionEnded`

```json
{
  "session_id": "01HV9...",
  "started_at": "2026-04-20T07:30:00Z",
  "location": "backyard",
  "mood": "focused"
}
```

```json
{
  "session_id": "01HV9...",
  "ended_at": "2026-04-20T08:05:00Z",
  "progress": { "pages_read": 12, "page_count": 1171 },
  "note": "intro chapter"
}
```

**Exception to 1:1 naming:** the `Progress` value object is
represented inline as a discriminated JSON object. Type discriminator
is the presence of `pages_read` (paper/ebook) vs.
`minutes_listened` (audiobook). The model talks about `Progress`;
the wire shape inlines the variant.

### `ReviewWritten` / `ReviewEdited`

```json
{
  "rating": 5,
  "body_path": "reviews/978-0-13-468599-1.md",
  "tags": ["architecture", "design"]
}
```

Note that `body` is _not_ in the event payload. The markdown file at
`body_path` is canonical. The event records the metadata only.

## Review file format

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

**Frontmatter rules:**

- `book_id` and `rating` are required.
- `written_at` is required and must parse as RFC 3339.
- `tags` is optional; defaults to `[]`.
- Unknown frontmatter keys are preserved on rewrite — the user can
  add their own.

A file with invalid frontmatter is **ignored** by the projection
and **flagged** by `bookshelf doctor`. The corresponding `Review`
is treated as missing until the file is fixed.

## Schema migrations

Additive only. Each migration is a numbered SQL file in
`migrations/NNN_description.sql`. On startup, `bookshelf` applies any
pending migration in order and records it in `schema_migrations
(version, applied_at)`. Rollbacks are out of scope; a wrong migration
is fixed by a new forward migration.

View tables (`books`, `sessions`, `reviews`) are eligible for
`DROP TABLE; CREATE TABLE; INSERT FROM events` rebuild during a
migration — the `events` table is the only source of truth.
