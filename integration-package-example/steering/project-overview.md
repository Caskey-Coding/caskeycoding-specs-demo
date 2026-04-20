# link-audit — Project Overview

**Status:** `steering` (always-inject)
**Last updated:** 2026-04-20

> Steering docs describe the project at the level of "things every session
> needs to know." They rarely change. When they do, it's an event —
> announce it in the next PR that touches the file.

## Mission

Make broken documentation cheap to find and expensive to merge. Every
markdown repo with more than a handful of cross-references eventually
drifts — link-audit catches the drift in CI, not in a user's browser.

## Users

- **Doc-heavy OSS maintainers** — want a green check on their PRs that
  proves no link rotted.
- **Internal platform teams** — run it over a private docs monorepo on
  a schedule.
- **Content writers** — run it locally before opening a PR so the review
  is about the words, not the mechanics.

## What "done" looks like for v1

- `link-audit check .` exits non-zero if any `.md` file under the current
  directory contains a broken link or a missing anchor.
- `link-audit check --format json` emits a machine-readable report.
- GitHub Action wraps the CLI, posts a PR comment with the diff of
  newly-broken vs. still-broken vs. newly-fixed links.
- Runs on a 500-file corpus in under 30 seconds on a free-tier GitHub
  runner.
- Zero false positives on the reference corpus (see
  `feature/001-broken-link-detection/spec.md §Acceptance`).

## What v1 explicitly does NOT do

- No rendering of markdown to HTML — that's a different tool's job.
- No autocorrect / autofix — the tool reports, humans decide.
- No crawling beyond the repo — external links are HEAD-checked, not
  spidered.
- No authoring UX, no web UI, no SaaS dashboard. CLI only.

## Success criteria (measurable)

- Adopted by at least 3 OSS repos within 6 months of v1 release.
- P50 CI wall-clock added by the action: < 15 seconds on a 500-file repo.
- < 5 issues opened about false positives in the first 90 days.

## Non-goals that sound like goals

- **Not a markdown linter.** There are already good ones. link-audit
  cares only about cross-references.
- **Not a spellchecker.** Same reason.
- **Not a replacement for human review.** Green CI means no broken
  links; it does not mean the docs are good.
