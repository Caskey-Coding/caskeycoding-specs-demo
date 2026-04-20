# ADR-002: SARIF as the Default Output Format

**Status:** `accepted`
**Date:** 2026-04-18
**Author:** Eric Caskey (demo author)
**Amends:** `feature/001-broken-link-detection/requirements.md` v1 (which shipped with text as default)

## Context

`link-audit` v0.1 shipped with three reporters: `text`, `json`, `sarif`.
The default was `text`. This decision predates the GitHub Action
wrapper (feature/002) and came from the "a human runs the CLI"
assumption.

Once feature/002 went into design, it became clear that the primary
consumer is not a human at a terminal — it's a CI pipeline, and that
pipeline wants SARIF so results land in the GitHub Security tab
automatically via `github/codeql-action/upload-sarif`.

Keeping `text` as the default means every CI consumer has to remember
`--format sarif`. A missed flag = silent loss of security-tab
integration. That's the wrong default.

## Decision

SARIF becomes the default output format. `text` and `json` remain
available via `--format text` / `--format json`.

CLI behavior under TTY detection: if stdout is a terminal (detected via
`isatty`), the CLI emits a one-line hint on stderr pointing at
`--format text` for human-friendly output. The default behavior —
SARIF to stdout — is unchanged regardless of TTY. We do not infer
format from TTY; inference hides the default from users who scripted
against it.

## Rationale

- Primary consumer is CI, not interactive shell.
- SARIF output is lossless against the internal `Report` model; it
  can be re-rendered to text or json downstream if needed.
- Existing v0.1 consumers are few (pre-1.0), so the migration cost is
  small, and this ADR ships before the 1.0 tag.

## Consequences

### Positive

- Zero-config GitHub code-scanning integration — drop the Action in,
  security tab populates.
- Machine-readable-first aligns with the product's CI orientation
  (see `steering/project-overview.md`).

### Negative

- Running `link-audit check .` at a terminal now dumps JSON-ish SARIF
  to stdout, which is verbose. The TTY hint mitigates the
  discoverability cost.
- Pre-1.0 text-default consumers must add `--format text`. Called out
  in the next release notes.

### Neutral

- Internal `Report` model and classifier are unchanged; this ADR only
  touches the default reporter selection.

## What this ADR supersedes

The original `feature/001` spec's §API section said:

> `--format text|json|sarif`. Default: text.

After this ADR, that sentence is incorrect. The requirements document
has been updated to say:

> `--format sarif|json|text`. Default: sarif. See ADR-002.

The change log in `feature/001-broken-link-detection/requirements.md`
records this amendment.

## Alternatives rejected

| Alternative | Reason rejected |
|-------------|-----------------|
| Keep text as default, document sarif prominently | Relies on users reading docs; the default is what actually ships |
| Infer format from TTY | Hides the default; breaks scripts that redirect stdout |
| Make json the default | Compromise between text and sarif but satisfies neither consumer well |

## Change log

| Date | Change |
|------|--------|
| 2026-04-18 | ADR created; SARIF becomes default; feature/001 requirements updated in the same PR |
