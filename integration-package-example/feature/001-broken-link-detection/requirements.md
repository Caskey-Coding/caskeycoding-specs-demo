# Feature 001 — Broken Link Detection

**Status:** `in-progress`
**Author:** Eric Caskey (demo author)
**Created:** 2026-03-14
**Updated:** 2026-04-20

## Summary

Scan every markdown file under a root path, extract every link, and
classify each as `ok`, `broken`, or `unreachable`. Report the results
in one of three formats (`sarif`, `json`, `text`) and exit non-zero if
any link is `broken`.

This is the core capability of `link-audit`. Everything else in the
project (the GitHub Action, the reporters, the config loader) exists to
serve this feature.

## Motivation

Documentation drifts. A rename in week 3 breaks a link written in
week 1 and nobody notices until a user lands on a 404. CI is the
cheapest place to catch that drift — much cheaper than an issue filed
a month later.

## Requirements

- [x] Walk a directory tree; parse every `.md` file via `goldmark`.
- [x] Extract internal links (relative paths, fragment-only) and
      external links (absolute URLs).
- [x] Classify internal links: resolve to a filesystem path; if the
      file is missing, or the `#anchor` doesn't exist, mark `broken`.
- [ ] Classify external links: issue HTTP HEAD with a 10s timeout;
      fall back to GET on 405. 2xx/3xx → `ok`, 4xx/410 → `broken`,
      5xx/timeout/DNS failure → `unreachable`.
- [ ] Retry `unreachable` once with exponential backoff before
      reporting.
- [ ] Respect a `.link-audit.yaml` config: allowlist patterns, ignored
      paths, concurrency ceiling, per-call timeout.
- [x] Three reporters: `sarif` (default), `json`, `text`.
      See [ADR-002](../../decision/ADR-002-sarif-as-default-output-format.md).

## API / Interface

### CLI

```
link-audit check <path> [--format sarif|json|text]          # default: sarif (ADR-002)
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
    File   string  // path to the .md file
    Line   int     // 1-indexed
    Link   string  // the raw link text
    Target string  // resolved target (path or URL)
    Status Status  // ok | broken | unreachable
    Detail string  // human-readable reason
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
  - "^https://example\\.internal/.*"   # intranet, not checkable from CI
timeout: 10s
retries: 1
```

## Data model

No persistent storage. The in-memory `Report` is the only data
structure. Reporters in `internal/report/{text,json,sarif}.go`
serialize it to stdout.

## Acceptance criteria

- [ ] On a 500-file reference corpus, `link-audit check` completes in
      under 30 seconds on a `ubuntu-latest` GitHub runner at
      concurrency 16.
- [ ] Zero false positives on the corpus (every link flagged `broken`
      is actually broken).
- [ ] `--format sarif` output passes GitHub code-scanning validation.
- [ ] A broken internal link in a nested subdirectory produces a
      `LinkResult` with the correct file path, line number, and
      resolved target.
- [ ] A 5xx response retries exactly once; a subsequent 5xx reports
      `unreachable`, not `broken`.
- [ ] `.link-audit.yaml` `allow_patterns` suppresses a matching URL
      entirely — it does not appear in the report at all.

## Out of scope

- Link autofix or rename-aware link updating.
- HTML file scanning.
- Headless-browser rendering for SPA targets.
- GitHub PR comment formatting — owned by [feature/002](../002-github-action-wrapper/requirements.md).

## Open questions

- Should `#` anchor resolution honor the goldmark heading-slug
  algorithm, or should we match GitHub's? **Resolved 2026-04-02:**
  GitHub's. See `architecture.md §Key Architectural Decisions`.
- Should we follow 3xx redirects before classifying? **Open.** Leaning
  yes with a max redirect depth of 5; pending a real-world corpus to
  validate.

## Implementation Reality

Pull-throughs of what is and isn't built — kept in sync with the
package-level `README.md §Implementation Reality`. Update both at
each reconciliation.

- External-link HTTP classification — not yet wired. The classifier
  short-circuits external URLs to `ok` in v0.1.
- SARIF reporter — skeleton exists; returns `ErrNotImplemented`.
- Config loader — not built; all settings are flags-only.

## Change log

| Date | Change |
|------|--------|
| 2026-03-14 | Requirements drafted; text-default reporter |
| 2026-04-02 | Anchor slug algorithm resolved to GitHub-compatible |
| 2026-04-18 | Default reporter changed to SARIF per [ADR-002](../../decision/ADR-002-sarif-as-default-output-format.md); §API and §Requirements updated |
| 2026-04-20 | Split package-scoped ADRs out of this feature into `decision/`; cross-references updated |

## References

- [Package README](../../README.md)
- [`steering/architecture.md`](../../steering/architecture.md)
- [ADR-001: Go as the implementation language](../../decision/ADR-001-go-as-implementation-language.md)
- [ADR-002: SARIF as the default output format](../../decision/ADR-002-sarif-as-default-output-format.md)
- Sibling feature: [GitHub Action wrapper](../002-github-action-wrapper/requirements.md)
