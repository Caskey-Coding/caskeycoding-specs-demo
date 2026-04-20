# Feature 001 — Broken Link Detection

**Status:** `in-progress`
**Author:** Eric Caskey
**Created:** 2026-03-14
**Updated:** 2026-04-20

## Summary

Scan every markdown file under a root path, extract every link, and
classify each as `ok`, `broken`, or `unreachable`. Report the results
in one of three formats (text, JSON, SARIF) and exit non-zero if any
link is `broken`.

This is the core capability of `link-audit`. Everything else in the
project (the GitHub Action, the reporters, the config loader) exists to
serve this feature.

## Motivation

Documentation drifts. A rename in week 3 breaks a link written in week 1
and nobody notices until a user lands on a 404. CI is the cheapest place
to catch that drift — much cheaper than an issue filed a month later.

## Requirements

- [x] Walk a directory tree; parse every `.md` file via `goldmark`.
- [x] Extract internal links (relative paths, fragment-only) and external
      links (absolute URLs).
- [ ] For each internal link: resolve to a filesystem path. If the file
      is missing, or the `#anchor` doesn't exist, classify `broken`.
- [ ] For each external link: issue an HTTP HEAD with a 10s timeout.
      Classify `ok` on 2xx/3xx, `broken` on 4xx/410, `unreachable` on
      5xx/timeout/DNS failure.
- [ ] Retry `unreachable` once with exponential backoff before reporting.
- [ ] Respect a `.link-audit.yaml` config: allowlist patterns, ignored
      paths, concurrency ceiling.

## API / Interface

### CLI

```
link-audit check <path> [--format text|json|sarif]
                       [--config .link-audit.yaml]
                       [--concurrency N]
                       [--log-level debug|info|warn|error]
```

Exit codes per `steering/conventions.md §CLI UX`.

### Go API (public)

```go
// Auditor scans a directory tree and produces a Report.
type Auditor interface {
    Audit(ctx context.Context, root string) (Report, error)
}

// Report is the result of one audit pass.
type Report struct {
    ScannedFiles int
    Links        []LinkResult
    Summary      Summary
}

// LinkResult is the classification of one link occurrence.
type LinkResult struct {
    File      string   // path to the .md file
    Line      int      // 1-indexed
    Link      string   // the raw link text
    Target    string   // resolved target (path or URL)
    Status    Status   // ok | broken | unreachable
    Detail    string   // human-readable reason
}

type Status string
const (
    StatusOK          Status = "ok"
    StatusBroken      Status = "broken"
    StatusUnreachable Status = "unreachable"
)
```

### Config file shape

```yaml
# .link-audit.yaml
concurrency: 16
ignore_paths:
  - "node_modules/**"
  - "vendor/**"
allow_patterns:
  - "^https://example\\.internal/.*"   # intranet link, not checkable from CI
timeout: 10s
retries: 1
```

## Data Model

No persistent storage. The in-memory `Report` is the only data
structure. Renderers in `internal/report/{text,json,sarif}.go`
serialize it to stdout.

## Acceptance Criteria

- [ ] On a 500-file reference corpus, `link-audit check` completes in
      under 30 seconds on a `ubuntu-latest` GitHub runner at concurrency
      16.
- [ ] Zero false positives on the corpus (every link flagged `broken`
      is actually broken).
- [ ] `--format sarif` output passes GitHub code-scanning validation.
- [ ] A broken internal link in a nested subdirectory produces a
      `LinkResult` with the correct file path, line number, and resolved
      target.
- [ ] A 5xx response retries exactly once; a subsequent 5xx reports
      `unreachable`, not `broken`.
- [ ] `.link-audit.yaml` `allow_patterns` suppresses a matching URL
      entirely — it does not appear in the report at all.

## Out of Scope

- Link autofix or rename-aware link updating.
- HTML file scanning.
- Headless-browser rendering for SPA targets.
- GitHub PR comment formatting — that lives in the Action, which is a
  separate future feature.

## Open Questions

- Should `#` anchor resolution honor the goldmark heading-slug algorithm,
  or should we match GitHub's? **Decided 2026-04-02:** GitHub's. See
  `decision-ADR.md §ADR-1`.
- Should we follow 3xx redirects before classifying? **Open —** leaning
  yes, but then we need a max redirect depth.

## References

- `steering/project-overview.md`
- `steering/tech-stack.md`
- `decision-ADR.md` (package-scoped ADR in this feature directory)
