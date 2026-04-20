# Feature 002 — GitHub Action Wrapper

**Status:** `draft`
**Author:** Eric Caskey (demo author)
**Created:** 2026-04-20
**Updated:** 2026-04-20

> Pedagogical note: this is a second feature in the same package. The
> point of including it is to show _feature accumulation_ — integration
> packages grow one feature directory at a time, with numbered
> subdirectories. Compare with `feature/001-broken-link-detection/`
> which is further along (status `in-progress`) while this one is at
> `draft`. In a real package, you'd see many features at different
> stages.

## Summary

Wrap the `link-audit` CLI as a GitHub composite action. On pull
requests, run the CLI against the PR's head ref, diff the report
against the base ref's report, and post a single PR comment with
three sections: newly-broken, still-broken, newly-fixed.

## Motivation

The CLI produces SARIF (per [ADR-002](../../decision/ADR-002-sarif-as-default-output-format.md))
and exits non-zero on `broken`. That's enough to _fail_ a CI check,
but not enough to show contributors which links their PR introduced
or fixed. The PR comment is the human-readable surface that turns a
red check into an actionable one.

## Requirements

- [ ] Composite action at `action/action.yml` that consumes a
      pre-built `link-audit` binary.
- [ ] Runs the CLI twice: once against the PR base, once against the
      PR head.
- [ ] Diffs the two reports: produces `newly-broken`, `still-broken`,
      `newly-fixed`.
- [ ] Posts a single PR comment. Subsequent runs edit the same comment
      (no comment spam). Uses a hidden marker HTML comment to locate
      the prior comment.
- [ ] Uploads the PR-head SARIF to GitHub code-scanning via
      `github/codeql-action/upload-sarif`.
- [ ] Exits non-zero iff `newly-broken` is non-empty. `still-broken`
      alone does not fail CI (it's the base ref's problem).

## Design sketch

```
    PR push → GH Actions                 (triggered)
             │
             ▼
    ┌───────────────────────┐
    │ caskey-coding/link-audit  │
    │   composite action       │
    └───────┬───────────────┘
            │
    ┌───────┴──────────┐
    │ checkout PR base   │
    │ link-audit --json  ├──▶  base.json
    └──────────────────┘
    ┌──────────────────┐
    │ checkout PR head   │
    │ link-audit --sarif ├──▶  head.sarif (uploaded)
    │ link-audit --json  ├──▶  head.json
    └─────────┬─────────┘
              ▼
    ┌──────────────────┐
    │ diff base ↔ head  ├──▶  PR comment (edit-in-place)
    └──────────────────┘
```

## Inputs

```yaml
inputs:
  paths:
    description: "Space-separated paths to scan"
    required: false
    default: "."
  config:
    description: "Path to .link-audit.yaml"
    required: false
  comment:
    description: "Whether to post/edit a PR comment"
    required: false
    default: "true"
```

## Outputs

```yaml
outputs:
  newly-broken-count:
    description: "Number of links broken by this PR"
  still-broken-count:
    description: "Number of links already broken on base"
  newly-fixed-count:
    description: "Number of links fixed by this PR"
```

## Acceptance criteria

- [ ] First run on a PR creates a new comment.
- [ ] Second run on the same PR edits the existing comment rather than
      posting a new one.
- [ ] Deleting all broken links edits the comment to say "all clear".
- [ ] Exits non-zero when `newly-broken-count > 0` regardless of
      `still-broken-count`.

## Out of scope

- Non-PR event types (push to main, schedule). These get SARIF upload
  only, no comment.
- Replying in thread to a broken-link comment with an autofix. That's
  an unrelated feature.
- Slack / Teams notifications. The PR comment is enough.

## Open questions

- Rate-limit behavior when the base ref has hundreds of already-broken
  links: truncate the comment or paginate with a "show more" link?
  **Open.**
- Should the action cache the base-ref report between runs to skip the
  double-scan on unchanged bases? **Open; probably yes via
  `actions/cache` keyed by base SHA.**

## Dependencies

- [feature/001](../001-broken-link-detection/requirements.md) must
  ship external-link classification and the SARIF reporter before this
  feature can be built. Until then, this spec stays `draft`.
- A published `link-audit` binary on GitHub Releases.

## References

- [Package README](../../README.md)
- [Sibling feature: Broken Link Detection](../001-broken-link-detection/requirements.md)
- [ADR-002: SARIF as default output format](../../decision/ADR-002-sarif-as-default-output-format.md)
