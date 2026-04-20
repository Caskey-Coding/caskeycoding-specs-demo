# ADR-001: Go as the Implementation Language

**Status:** `accepted`
**Date:** 2026-03-01
**Author:** Eric Caskey (demo author)

## Context

`link-audit` is a CI tool. It runs in GitHub-hosted Linux runners, in
self-hosted runners on arm64, on developer laptops (macOS arm64), and
occasionally on Windows. It needs to start fast (CI latency is visible
to every user on every PR), fetch many URLs concurrently (network is
the bottleneck), and ship as a single file because CI environments
rarely have a package manager installed.

Candidates surveyed:

- **Go 1.22.x** — static binary per platform, strong stdlib HTTP
  client, `goroutines` for concurrent link checks, first-class GitHub
  Actions ecosystem.
- **Node.js / TypeScript** — ubiquitous in CI but requires a Node
  runtime on the runner or bundling via `pkg` / `ncc` (fragile at
  scale).
- **Rust** — same single-binary strengths, but with a steeper
  contributor ramp and slower iteration on a small tool.
- **Python** — great for local dev, painful for distribution (requires
  a Python runtime; `pyinstaller` binaries are large and slow to
  start).

## Decision

Implement `link-audit` in Go 1.22.x. Pin the minor version. An upgrade
requires an amendment to this ADR.

## Rationale

Of the candidates, Go is the only one that satisfies all four
constraints (cold-start speed, native concurrency for HTTP, single-file
distribution, low contributor ramp) without bespoke tooling. The GitHub
Action wrapper (feature/002) is a composite action that embeds the
binary; Node would require a JavaScript action with the Go binary
shelled out — more moving parts for the same result.

## Consequences

### Positive

- Single binary per OS/arch via `goreleaser`.
- No runtime dependency on the consumer's environment.
- Concurrency via `errgroup` is idiomatic and readable.
- Large standard library — HTTP, URL parsing, filepath walking, all
  stdlib.

### Negative

- Markdown parsing library (`goldmark`) is less battle-tested than the
  Node ecosystem's options.
- Contributors more familiar with JS/TS need a Go ramp, but Go's
  learning curve for this shape of tool is shallow.

### Neutral

- Release pipeline uses `goreleaser` — one more tool in the chain, but
  it's the standard for Go CLI distribution.

## Alternatives rejected

| Alternative | Reason rejected |
|-------------|-----------------|
| Node.js + `ncc` bundling | Runtime-dependent; large bundle; slow cold start in CI |
| Rust | Higher contributor ramp for a small tool; slower iteration |
| Python + `pyinstaller` | Binary size and cold-start cost unacceptable for CI |
| Pure shell + `curl` | Cannot parse markdown to extract links correctly |

## Change log

| Date | Change |
|------|--------|
| 2026-03-01 | ADR created; Go 1.22.x pinned |
