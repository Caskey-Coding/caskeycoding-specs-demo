# bookshelf — Spec Package

**Status:** `draft`
**Author:** Eric Caskey (demo author)
**Created:** 2026-04-20
**Updated:** 2026-04-20

> **This is a fictional pedagogical example.** `bookshelf` is not a
> real product. This spec package exists to demonstrate the shape of a
> _domain package_ as used for real work in the private
> `caskeycoding-specs` repo. The framework is governed by
> [ADR-001 (Spec-Driven Development)](../decision/ADR-001-spec-driven-development.md)
> and [ADR-002 (canonical SDD file structure)](../decision/ADR-002-sdd-file-structure.md).
> Read this README first, then follow the links to see how product,
> domain, contracts, and delivery fit together.

## Summary

`bookshelf` is a single-user CLI plus local web UI for tracking books
you are reading, have read, and have abandoned. It is local-first —
SQLite plus a directory of markdown review files — with no cloud
component and no multi-tenancy. The whole product is a `~/.bookshelf/`
directory and a single Go binary.

This is a domain package because the product has a clear bounded
scope, a stable domain model (Book ↔ ReadingSession ↔ Review), explicit
contracts (CLI verbs, local HTTP API, file format), and a finite
delivery plan. There is no platform to integrate into; the surface is
the filesystem and the user. Compare with the
[integration-package example](../integration-package-example/README.md),
which evolves alongside an external CI ecosystem.

## Spec Index

```
domain-package-example/
├── README.md                          ← you are here
│
├── product/                           What and why
│   ├── 001-requirements.md            Goals, users, scope, success criteria
│   └── 002-use-cases.md               Narrative flows: the weekly review,
│                                      the abandoned-book check, the pattern
│
├── domain/                            Core model and business rules
│   ├── 001-model.md                   Entities, value objects, invariants,
│   │                                  state transitions
│   └── 002-data-contracts.md          SQLite schema, review file frontmatter,
│                                      on-disk layout under ~/.bookshelf/
│
├── contracts/                         Interfaces between components
│   └── 001-api.md                     CLI verbs and local HTTP API
│
└── delivery/                          How we build, test, and ship
    ├── 001-acceptance-tests.md        Test scenarios, coverage targets
    └── 002-rollout-plan.md            Phases, observability, risks
```

Reading order for a new contributor: README → `product/001` →
`domain/001` → `contracts/001` → `delivery/002`. Everything else is
reference.

## Architecture

```
┌────────────────────────────────────────────────────────────────────────┐
│                              bookshelf                              │
├──────────────┬────────────────┬───────────────┬─────────────────┤
│ CLI Layer    │ Domain Core    │ Persistence   │ Web UI            │
│ (cobra)      │ (pure Go)      │ (SQLite +     │ (`bookshelf web`)│
│              │                │  markdown FS) │ (loopback only)  │
├──────────────┼────────────────┼───────────────┼─────────────────┤
│ add | start  │ Book           │ sqlite3       │ GET /api/books    │
│ end | status │ ReadingSession │ (data.sqlite) │ POST /api/books   │
│ review       │ Review         │ reviews/*.md  │ PATCH /api/...    │
│ query | q    │ Events:        │ config.yaml   │ PUT /api/.../review
│ web | doctor │   BookAdded    │ schema_migs   │                   │
│              │   StatusChanged│               │                   │
│              │   SessionStart │               │                   │
│              │   SessionEnded │               │                   │
└──────────────┴────────────────┴───────────────┴─────────────────┘
```

### Data Flow

```
user intent ($EDITOR, terminal, browser)
        │
        ▼
  ┌──────────┐      ┌───────────┐      ┌────────────┐      ┌──────────┐
  │ CLI verb ├─────▶│ Domain  ├─────▶│ Event store├─────▶│ SQLite + │
  │ (cobra)  │      │ service │      │ (append)   │      │ markdown │
  └──────────┘      └──────────┘      └────────────┘      └──────────┘
        │                 │
        │                 ▼
        │           ┌───────────┐
        │           │ Materialized│  ← read-side projection
        │           │   views     │     for `query` subcommand
        │           └─────┬─────┘     and HTTP GETs
        │                 │
        ▼                 ▼
  stdout / stderr        HTTP JSON
  (text or JSON)         (web UI on 127.0.0.1)
```

## Key Architectural Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Single tenant | Yes | One user, one machine, one `~/.bookshelf/` |
| Storage model | Append-only events + materialized views | Audit + replay; SQLite handles both trivially |
| Persistence | SQLite + plain markdown reviews | Hand-editable; survives the tool |
| Web UI scope | Loopback only | No auth needed; not a hosted product |
| Sync between machines | Out of scope | Files in `~/.bookshelf/` are user's responsibility |
| Format ratings | Integer 1–5, no half-stars | Avoids the "4.5 but I wanted 5" debate |
| Audiobook progress | Minutes-listened, not chapters | Chapters vary; minutes are universal |

## Module Structure

```
bookshelf/
├── cmd/
│   └── bookshelf/
│       └── main.go             # cobra root + subcommand wiring
├── internal/
│   ├── domain/                 # entities, value objects, invariants
│   │   ├── book.go
│   │   ├── session.go
│   │   ├── review.go
│   │   └── events.go
│   ├── store/                  # event store + projections
│   │   ├── sqlite.go
│   │   ├── migrations/
│   │   └── projections.go
│   ├── reviews/                # markdown frontmatter parser/writer
│   │   └── markdown.go
│   ├── cli/                    # cobra command handlers
│   │   ├── add.go
│   │   ├── start.go
│   │   ├── end.go
│   │   ├── status.go
│   │   ├── review.go
│   │   ├── query.go
│   │   └── web.go
│   └── webui/                  # local HTTP server + bundled assets
│       ├── server.go
│       └── static/             # bundled at build time
└── go.mod
```

## Out of Scope (MVP)

- Sync between machines.
- Multi-user accounts.
- Public API (anything beyond loopback HTTP).
- Mobile native clients.
- Import from Goodreads / StoryGraph / LibraryThing.
- Backups (the user owns the file).
- Recommendations or ML on reading patterns.
- A SaaS dashboard.

## References

- Framework: [ADR-001: Spec-Driven Development](../decision/ADR-001-spec-driven-development.md)
- File structure: [ADR-002: Canonical SDD File Structure](../decision/ADR-002-sdd-file-structure.md)
- Sibling archetype: [Integration Package Example](../integration-package-example/README.md)
