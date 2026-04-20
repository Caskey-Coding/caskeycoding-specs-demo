# link-audit — Architecture

**Status:** `steering` (always-inject)
**Last updated:** 2026-04-20

> Steering doc. Read at the start of every session. This is the
> always-true shape of the system; mid-feature details live in
> `feature/<n>/requirements.md`.

## System Overview

```
┌────────────────────────────────────────────────────────────────────────┐
│                           link-audit                                │
├──────────────┬───────────────┬────────────────┬──────────────────┤
│ CLI Entrypoint│ Scanner       │ Classifier     │ Reporters         │
│ (cobra)      │ (walk + parse)│ (resolve +     │ text | json      │
│              │               │  HTTP check)   │                  │
│              │               │                │ sarif (ADR-002)  │
├──────────────┼───────────────┼────────────────┼──────────────────┤
│ check | list │ goldmark AST  │ HEAD w/ 405    │ stdout or file   │
│ version      │ per-file      │  fallback GET  │                  │
│              │ line mapping  │ anchor resolver│                  │
└──────────────┴───────────────┴────────────────┴──────────────────┘
```

## Data Flow

```
markdown ──┐      ┌──────────┐     ┌──────────────┐     ┌────────────┐
files      ├───▶ │  Scanner  ├──▶ │  Classifier   ├──▶ │  Report    │
(.md)      │     │  (walk)   │     │  (per-link)  │     │  (in-mem)  │
           │     └──────────┘     └──────┬───────┘     └─────┬──────┘
.link-audit│                             │                   │
.yaml ─────┘                    ┌────────┴────────┐          │
                                │ internal: fs    │          │
                                │ external: HTTP  │          │
                                └─────────────────┘          ▼
                                                      ┌─────────────┐
                                                      │  Reporter   │
                                                      │ text / json │
                                                      │    / sarif  │
                                                      └─────────────┘
                                                             │
                                                             ▼
                                                          stdout
                                                       (+ exit code)
```

## Key Architectural Decisions

| Decision | Choice | Rationale | ADR |
|----------|--------|-----------|-----|
| Implementation language | Go 1.22.x | Single-binary distribution, stdlib HTTP, GH Action support | [ADR-001](../decision/ADR-001-go-as-implementation-language.md) |
| Default output format | SARIF | GitHub code-scanning consumes it natively | [ADR-002](../decision/ADR-002-sarif-as-default-output-format.md) |
| Markdown parser | `goldmark` | Pure-Go, CommonMark-compliant, fast | tech-stack.md |
| External HTTP method | HEAD, fall back to GET on 405 | Cheap common case, correct tail | feature/001 decision |
| Anchor slug algorithm | GitHub's (not goldmark's) | Users target github.com renders | feature/001 decision |
| Retry policy | Exactly one retry on transient failure | Balances CI latency and flakiness | feature/001 decision |
| Persistence | None | Reports are files on stdout; tool is stateless | — |

## Module Structure

```
link-audit/
├── cmd/
│   └── link-audit/
│       └── main.go                  # cobra root + subcommand wiring
├── internal/
│   ├── scanner/
│   │   ├── walk.go                  # filepath.WalkDir + .md filter
│   │   └── parse.go                 # goldmark AST → raw link list
│   ├── classifier/
│   │   ├── internal.go              # fs path resolution + anchor check
│   │   ├── external.go              # HTTP HEAD w/ 405 → GET fallback
│   │   └── slug.go                  # GitHub-compatible heading slug
│   ├── report/
│   │   ├── text.go                  # human-readable
│   │   ├── json.go                  # machine-readable
│   │   └── sarif.go                 # GitHub code-scanning (ADR-002)
│   └── config/
│       └── loader.go                # .link-audit.yaml parser
├── test/
│   └── integration/
│       ├── fixtures/                # sample markdown corpora
│       └── server/                  # test HTTP server (405-on-HEAD, etc.)
├── action/                          # feature/002: GitHub composite action
│   ├── action.yml
│   └── post-comment.go
└── go.mod
```

## Integration surface

`link-audit` has no platform integration — it's a standalone tool.
Integration happens at the consumer's CI, via:

| Consumer surface | How |
|------------------|-----|
| Shell pipeline | `link-audit check .` → exit code |
| GitHub Actions | `caskey-coding/link-audit-action@v1` (feature/002) |
| SARIF upload | Generated report uploaded via `github/codeql-action/upload-sarif` |
| Pre-commit hook | Standalone binary invocation |

## Non-architecture (what NOT to design in)

- No daemon. No long-running process. The tool exits after reporting.
- No state file. No cache across runs. Each invocation is fresh.
- No network calls other than the link checks themselves — no
  telemetry, no update checks, no phone-home.
- No plugins. A new classifier or reporter is a new internal package,
  not a dynamically-loaded shared object.
