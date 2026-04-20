# link-audit — Spec Package

**Status:** `draft`
**Author:** Eric Caskey (demo author)
**Created:** 2026-04-20
**Updated:** 2026-04-20

> **This is a fictional pedagogical example.** `link-audit` is not a
> real project. This spec package exists to demonstrate the shape of an
> _integration package_ (per [ADR-001](../decision/ADR-003-spec-driven-development.md)
> and [ADR-004](../decision/ADR-004-sdd-file-structure.md)) as used for
> real work in the private `caskeycoding-specs` repo. Read this README
> first, then follow the links to see how steering, features, and
> decisions fit together.

`link-audit` is a fictional CLI + GitHub Action that scans markdown
repositories for broken links, missing anchors, and stale references.
Written in Go, shipped as a static binary and a reusable composite
action. It integrates into CI pipelines — which is what makes it an
_integration package_: requirements will evolve as real-world CI
systems surface edge cases.

---

## Directory Map

| Directory | Purpose | When to read |
|-----------|---------|--------------|
| [`steering/`](./steering/) | Project-wide conventions, tech stack, architecture | Every session — always inject |
| [`feature/`](./feature/) | Scoped requirements per feature area | When implementing that feature |
| [`decision/`](./decision/) | ADRs — why a choice was made, what was rejected | When evaluating related changes |
| [`CLAUDE.md`](./CLAUDE.md) | Agent routing file that lives at the code-repo root | Every session (loaded by the code repo, not this spec package) |

## Steering Docs

- [Project Overview](./steering/project-overview.md) — Mission, users, success criteria
- [Conventions](./steering/conventions.md) — Code style, errors, tests, CLI UX
- [Tech Stack](./steering/tech-stack.md) — Go version pin, deps, distribution, CI
- [Architecture](./steering/architecture.md) — Components, data flow, module structure

## Feature Specs

- [Broken Link Detection](./feature/001-broken-link-detection/requirements.md) — Core capability; scans markdown, classifies links `ok` / `broken` / `unreachable`. Status: `in-progress`.
- [GitHub Action Wrapper](./feature/002-github-action-wrapper/requirements.md) — Wraps the CLI as a composite action; posts a PR comment with the diff of newly-broken vs still-broken vs newly-fixed links. Status: `draft`.

## Decision Records

- [ADR-001: Go as the implementation language](./decision/ADR-001-go-as-implementation-language.md) — Static-binary distribution + strong stdlib HTTP. `accepted`.
- [ADR-002: SARIF as the default output format](./decision/ADR-002-sarif-as-default-output-format.md) — GitHub code-scanning consumes SARIF natively. **Amends** `feature/001` v1 (which shipped text as default). `accepted`.

## Implementation Sequence

```
Phase 1: CLI core (feature/001)
         → Walk directory, parse markdown, classify internal + external links
         → Three reporters (text, json, sarif)
         → Unit + integration tests against a fixture server

Phase 2: GitHub Action wrapper (feature/002)
         → Composite action consumes the CLI binary
         → PR comment with newly-broken / still-broken / newly-fixed diff
         → Published to GitHub Marketplace

Phase 3: Configurability
         → .link-audit.yaml config (allowlist, ignore paths, concurrency)
         → Exit-code and retry tuning
         → SARIF schema version negotiation
```

## Agent Instructions

1. **Read all `steering/` docs before starting any implementation.** All
   four are always-inject. This includes `architecture.md`.
2. **When implementing a feature, read the matching `feature/<n>/requirements.md`.**
   Do not start from the spec package README alone.
3. **Before making architectural changes, check `decision/` for
   existing ADRs.** If a change contradicts an ADR, you need a new ADR
   that supersedes it — do not silently deviate.
4. **ADR-002 governs output defaults.** SARIF is the default format.
   Text is available via `--format text`. Do not reintroduce text as
   the default without superseding ADR-002.
5. **One spec, one PR.** If a change spans two features, split it into
   two PRs, each referencing its own spec.
6. **Update status and Implementation Reality on merge.** After
   merging, move the spec's `Status:` field forward in the lifecycle
   (`draft` → `ready` → `in-progress` → `completed` → `deprecated`)
   and note any deviations in the Implementation Reality section
   below.
7. **Every new external dependency is an ADR.** No silent additions
   to `go.mod`.

## Implementation Reality (2026-04-20)

> This section is authoritative for "what is actually built" and gets
> updated at every spec/code reconciliation. Pedagogical note: in a
> real package, this section is often the longest and most
> frequently-updated part of the README. The live `financial-reviewer`
> README dedicates more than half its length to Implementation Reality
> across multiple sprints. Here it's short because this is a demo.

**Last reconciliation:** 2026-04-20 — v0.1 walking skeleton.

### What shipped in v0.1

- `link-audit check <path> --format json` walks a tree, parses markdown
  via `goldmark`, and classifies internal links.
- Text reporter.
- Unit tests for the classifier.
- Fixture-server integration test (5 scenarios).

### Spec-but-not-in-code

These requirements are in `feature/001-broken-link-detection/requirements.md`
but no code implements them yet. They're pulled here so contributors
can see the gap at a glance.

- External-link HTTP HEAD checking — the classifier short-circuits
  external URLs to `ok` in v0.1.
- `.link-audit.yaml` config loader — all settings are flags-only in
  v0.1.
- SARIF reporter — spec'd in ADR-002 but the code path returns
  `ErrNotImplemented`.
- `feature/002-github-action-wrapper/` — not started; spec only.

### Dead config surfaces

- `--retries` flag is parsed but ignored (external-link checking
  isn't wired yet, so there's nothing to retry).
- `.link-audit.yaml` example file in `testdata/` references
  `allow_patterns:` — the loader doesn't exist yet, so this is
  documentation-only.

### Follow-up PRs this document anticipates

1. **External-link HEAD-with-405-GET-fallback** (closes spec
   `feature/001 §Requirements` external items; honors ADR-001's
   "tri-state result" convention).
2. **SARIF reporter** (closes ADR-002; unblocks GitHub code-scanning
   adoption).
3. **Config loader** (closes spec `feature/001 §Config file shape`).
4. **GitHub Action wrapper** (feature/002; blocked on #1–#3 because the
   Action needs JSON-producing CLI output).

---

## References

- Parent framework: [ADR-003: Spec-Driven Development](../decision/ADR-003-spec-driven-development.md)
- File structure: [ADR-004: Canonical SDD File Structure](../decision/ADR-004-sdd-file-structure.md)
- Sibling archetype: [Domain Package Example](../domain-package-example/README.md)
