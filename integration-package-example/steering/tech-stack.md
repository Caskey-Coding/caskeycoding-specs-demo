# link-audit — Tech Stack

**Status:** `steering` (always-inject)
**Last updated:** 2026-04-20

## Language

- **Go 1.22.x** — pinned minor. Upgrade requires an ADR. Chosen for
  single-binary distribution, strong stdlib HTTP client, and first-class
  GitHub Actions support.

## Direct dependencies

| Dependency | Purpose | Pinned | Rationale |
|-----------|---------|--------|-----------|
| `github.com/spf13/cobra` | CLI command tree | `v1.8.x` | Ubiquitous, battle-tested |
| `github.com/yuin/goldmark` | Markdown AST | `v1.7.x` | Fast, CommonMark-compliant, pure-Go |
| `golang.org/x/sync/errgroup` | Concurrent link checks | latest | Stdlib-adjacent |
| `github.com/stretchr/testify` | Assertions | `v1.9.x` | Test-only |

Any new direct dependency requires an ADR in
`feature/<name>/decision-ADR.md`.

## Runtime targets

- Linux x86_64 and arm64 (GitHub-hosted and self-hosted runners).
- macOS arm64 (developer machines).
- Windows x86_64 (community request — smoke-tested only).

## Distribution

- Static binary per OS/arch via `goreleaser`.
- GitHub Action wrapper: `caskey-coding/link-audit-action@v1`.
- Homebrew tap, scoop manifest — backlog; not in v1.

## CI pipeline

1. `go test ./... -race -coverprofile=cover.out`
2. `go tool cover -func=cover.out` — fail below 75% on `internal/`
3. `golangci-lint run`
4. `go build ./cmd/link-audit` for each target
5. Integration suite against the reference corpus in `test/integration/`
6. `goreleaser release --snapshot` on every PR; full release on tag

## Release

- Semver. Breaking changes to the CLI flag set = major.
- Action versions are re-tagged on patch (e.g. `v1` → `v1.2.3`).
- Release notes are auto-generated from conventional commits.

## Out of stack

- No database. Reports are files.
- No server. CLI only.
- No telemetry. Zero network calls except the link checks themselves.
- No authentication. The tool reads local files and performs
  unauthenticated HTTP requests.

## Infra

GitHub: source, releases, Action marketplace. That's the whole infra
surface. If we ever need more, it's a new ADR.
