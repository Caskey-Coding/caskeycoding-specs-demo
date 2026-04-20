# link-audit — Agent Instructions (code repo root)

> This is a **sanitized sample** of the `CLAUDE.md` file that sits at the root
> of a code repo — the first file any Claude Code or Cursor session reads. It
> routes the agent to the right specs for the task at hand. In the live
> caskeycoding.com codebases, this file lives at the root of each of the code
> repos (`ai-blog-web/CLAUDE.md`, `ai-blog-api/CLAUDE.md`, etc.).
>
> The repo, the tool, and the specs it points to are **fictional** — this is
> for pedagogical clarity, not a real project.

---

## What this repo is

`link-audit` is a CLI + GitHub Action that scans markdown repositories for
broken links, missing sections, and stale references. Written in Go, shipped
as a static binary and a reusable action.

## Spec package

All requirements, conventions, and decisions live in the spec package at
`../caskeycoding-specs-demo/integration-package-example/`. The agent MUST
load steering docs before touching any code.

## Routing table

| Task | Go to | Read first |
|------|-------|------------|
| Any code change | spec package | `steering/project-overview.md`, `steering/conventions.md`, `steering/tech-stack.md` |
| New feature | `feature/<name>/spec.md` | matching feature directory |
| Architectural pivot | `feature/<name>/decision-ADR.md` | existing ADRs in the feature |
| CI / release | `steering/tech-stack.md` §Release | — |
| Spec does not exist yet | write one first | `_template.md` in the parent specs repo |

## Hard rules

1. **Spec first.** Never write code without a matching spec. If a spec
   doesn't exist, write one before opening a code PR.
2. **Steering is always-inject.** Load all three `steering/*.md` docs at
   session start, every session, even for one-line fixes.
3. **ADRs are load-bearing.** If a change contradicts an ADR in
   `feature/*/decision-ADR.md`, you need a new ADR that supersedes it —
   do not silently deviate.
4. **One spec, one PR.** If a change spans two features, split it into
   two PRs, each referencing its own spec.
5. **Update status on merge.** After merging, move the spec's status from
   `in-progress` to `completed` and note any deviations in the
   Implementation Reality section of the package `README.md`.
6. **Test before you claim complete.** CI must be green. Manual smoke
   test on a sample markdown corpus must pass.

## What NOT to do

- Do not invent an API shape. The contracts are in
  `feature/<name>/spec.md §API`.
- Do not add a dependency without an ADR. Every dep is an ADR.
- Do not write to this `CLAUDE.md` without a spec change justifying it.

## Commit conventions

- Format: `type(scope): summary`
- Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`
- Scope matches the feature directory name when relevant

Examples:
- `feat(broken-link-detection): retry HTTP 429 with exponential backoff`
- `docs(steering): pin go 1.22.x`
