# link-audit — Conventions

**Status:** `steering` (always-inject)
**Last updated:** 2026-04-20

## Language and style

- Go 1.22.x (pinned — see `tech-stack.md`).
- `gofmt` + `goimports` on save. `golangci-lint run` clean before any PR.
- No vendored dependencies. `go mod tidy` on every change.
- Public API surface: only exported types and functions documented with
  a full sentence comment starting with the identifier name.

## Error handling

- Errors are values. Wrap with `fmt.Errorf("operation: %w", err)`.
- No `panic` outside of `main()` bootstrap failures.
- Tri-state for external results: `ok` / `transient` / `permanent`. A
  transient failure (HTTP 429, 5xx, timeout) retries with backoff; a
  permanent failure (HTTP 404, DNS NXDOMAIN) surfaces immediately.

## Naming

- Packages: one word, lowercase, no underscores. `linkcheck`, not
  `link_check` or `linkChecker`.
- Files: `snake_case.go`.
- Types: `CamelCase`. Interfaces end in `-er` when the role is a single
  verb (`Fetcher`, `Reporter`).

## Testing

- Unit tests for every exported function. Table-driven where the input
  space is enumerable.
- Integration tests live in `test/integration/` and hit a local fixture
  server, never the live internet.
- Coverage target: 80% on the `internal/` tree. CI fails below 75%.
- Golden-file tests for the JSON output format; regenerate with
  `go test ./... -update`.

## Logging

- Structured logs via `log/slog`. No `fmt.Println` in library code.
- Levels: `DEBUG` for per-link detail, `INFO` for file-level summary,
  `WARN` for transient failures, `ERROR` for permanent failures.
- CLI respects `--log-level` and `NO_COLOR`.

## CLI UX

- One verb per subcommand: `check`, `list`, `version`.
- Exit codes: `0` clean, `1` links broken, `2` usage error, `3` internal
  error.
- `--format text|json|sarif`. SARIF is what GitHub code-scanning
  consumes.

## Commits and PRs

- See `CLAUDE.md §Commit conventions`.
- PR title mirrors the commit subject.
- PR body links to the spec in `feature/<name>/spec.md`.
- Squash-merge. No merge commits on main.

## What NOT to do

- No global state. Config flows through constructor arguments.
- No `init()` side effects beyond registering the root cobra command.
- No third-party logging libraries. `log/slog` is enough.
- No reflection except in test helpers.
