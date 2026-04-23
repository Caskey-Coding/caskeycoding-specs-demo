# CI/CD Bar-Raiser Audit — `caskeycoding/caskeycoding-specs-demo`

**Audit date:** 2026-04-23
**Target default branch:** `main`
**Risk profile:** reputational (public-facing teaching artifact), not operational. No user data, no deployable artifact, no production surface.

> **Completeness note.** Audited inside the ~30 min time-box. All seven dimensions assessed. Branch protection APIs not queryable with the available tools — inferred from `list_branches` output (`"protected": false` on `main`) and flagged in §2.

---

## Executive summary

| # | Dimension | Score |
|---|-----------|-------|
| 1 | Correctness & reliability | **missing** — no CI exists |
| 2 | Security & supply chain | **gap** — no branch protection; sync trust model is informal; no secret-scanning policy declared |
| 3 | Speed & cost | **N/A** — no workflows, so nothing to measure |
| 4 | Deploy & release | **N/A** — repo ships no deployable artifact |
| 5 | Observability & feedback | **missing** — nothing runs, nothing is observed |
| 6 | Developer experience | **gap** — the "no CI" choice is defensible for a 20-file static repo, but one guardrail (NDA scan) is not optional given the repo's stated purpose |
| 7 | Config integrity | **missing** — no `.github/` directory at all |

### Top 3 must-fix (P0 / P1)

1. **P0 — No NDA content scan on PRs to a public repo whose entire reason for existing is "don't leak NDA content from the private specs repo"** (README:31–33, `.github/` absent). The upstream `pr-validation.yml:58–115` is the exact mitigation and doesn't exist here. One leaked internal term in a future curation pass and the reputational premise of the demo is inverted.
2. **P1 — `main` is unprotected** (`list_branches` → `"protected": false`). A direct push to `main` — including an accidental `git push --force` from a phone session — bypasses the review that the blog series ("Post 1 — Phone-First," README:151–152) explicitly celebrates.
3. **P1 — Sync from `caskey-coding/caskeycoding-specs` → `caskeycoding/caskeycoding-specs-demo` is entirely manual and undocumented** (README:79–82; no script, no workflow, no CODEOWNERS, no checklist found). ADR-006 governs intent but there is no codified process.

### Top 3 quick wins (< 1 hour each)

1. **Copy upstream `pr-validation.yml` verbatim, keep only `changed-files` + `nda-scan` + `pr-comment`.** Adapt base branch from `master` to `main`. ~15 minutes.
2. **Enable branch protection on `main`:** require PR, require the new NDA-scan check, disallow force-push, disallow deletion. Pure settings.
3. **Delete the stale `claude/populate-specs-demo-PL0Ph` branch** — merged in PR #1 three days ago.

---

## Phase 1 — Inventory

### Workflow files

| File | Triggers | Jobs | Notes |
|------|----------|------|-------|
| _(none)_ | — | — | `.github/` directory does not exist. |

### Deploy scripts / release automation

None. README:76–82 explicitly states "Not continuously synced … read-only artifact for humans arriving from the blog."

### External actions used

None.

### Branch protection

`main` → `"protected": false`. No required checks, no required reviewers, no restriction on force-push or deletion.

### Trigger → jobs → artifacts graph

```
(no triggers) → (no jobs) → (no artifacts, no deploys)
```

### Recent run history

Zero Actions runs possible. 20 commits in history, all authored 2026-04-20 within a ~75-minute window by a single committer, landed via one merge commit on `main` through PR #1.

---

## Phase 2 — Dimension review

### 1. Correctness & reliability — **missing**

- **Finding 1.1 — `Risk`** — Repo is a sync target from a private repo with no automated check that sync happened correctly. No diff gate, no structural-shape check. `decision/README.md:26–36` notes "ADR-002 reproduced verbatim from private's ADR-004 with `Depends on` header adjusted" — exactly the kind of manual edit that rots silently.

### 2. Security & supply chain — **gap**

- **Finding 2.1 — `Defect` — P0 — NDA content scan is missing.** Upstream `pr-validation.yml:58–115` runs a 14-term blocklist, org-name blocklist, attribution patterns, and a 12-digit AWS account-ID regex. None of this runs on the demo. The whole repo's thesis (README:9–13 — NDA-bound work stays private) is falsified by one leaked line.
- **Finding 2.2 — `Risk` — P1 — `main` branch unprotected.** Given public audience and phone-based edits, a single mis-targeted push alters public content with no review.
- **Finding 2.3 — `Risk` — P1 — Sync process not codified.** README:79–82 references private ADR-006 for governance, but no script/workflow/CODEOWNERS/checklist in-repo. Low-frequency, high-stakes manual work with no checklist = human-factors incident shape.
- **Finding 2.4 — `Opinion`** — No Dependabot/Renovate. N/A — no dependency manifests. Skipped.
- **Finding 2.5 — `Opinion`** — Zero `secrets.*` surface. `search_code` for `secrets.` → 0 results. Good.
- **Finding 2.6 — `Opinion`** — No `CODEOWNERS`. Cosmetic for a one-committer repo; useful once branch protection lands.

### 3. Speed & cost — **N/A**

No workflows. Skipping.

### 4. Deploy & release — **N/A**

Repo produces no deployable artifact. Rollback is `git revert`. All appropriate for a 20-file static teaching surface.

- Operational Lesson §2 config-pair drift: N/A — no code.
- §3 ruff pin: N/A — no Python.
- §4 lazy SDK imports: N/A — no code.
- §5 CDK stack graph: N/A — no infra.

### 5. Observability & feedback — **missing**

Nothing runs, so nothing reports. PR #1 merged without any automated signal — fine *if* the maintainer owns every risk the upstream NDA scan covers. See Finding 2.1.

### 6. Developer experience — **gap**

- **Finding 6.1 — `Opinion`** — No `make ci` / `npm run ci` local loop. A 10-line `make check` running `markdownlint-cli2` on staged files would close the local-vs-CI gap.
- **Finding 6.2 — `Risk`** — Stale branch `claude/populate-specs-demo-PL0Ph` not deleted. Enable "auto-delete head branches" in repo settings.

### 7. Config integrity — **missing**

No `.github/` means no `actionlint`, no reusable workflows, no issue/PR templates, no Dependabot config, no CODEOWNERS.

- **Finding 7.1 — `Risk` — P2** — No PR template. A brief template prompting "have you checked the NDA blocklist" documents the policy and backstops the scan.

---

## Findings table

| ID | Severity | File:line | Dimension | Finding | Recommendation | Effort |
|----|----------|-----------|-----------|---------|----------------|--------|
| 2.1 | **P0** | `.github/` absent (ref upstream `pr-validation.yml:58–115`) | Security | No NDA content scan on a public repo whose premise is "no NDA leakage" | Port upstream `nda-scan` job; base branch `master` → `main`; drop non-applicable path filters | S |
| 2.2 | **P1** | `list_branches` → `main.protected=false` | Security | `main` unprotected; direct push / force push possible | Enable branch protection: require PR, require NDA-scan check, disallow force-push + deletion, require linear history | S |
| 2.3 | **P1** | `README.md:79–82`; `decision/README.md:17` | Security | Sync from private → public is manual and undocumented in-repo | Add `_shared/sync-playbook.md` with step-by-step extraction checklist; commit block-term list alongside | M |
| 1.1 | P2 | `decision/README.md:26–36` | Correctness | ADR-002 is "verbatim from private with one header adjusted" — manual edit with no drift check | Optional CI job: fetch upstream ADR-004 body, diff vs local ADR-002 (excluding known delta) | M |
| 6.1 | P3 | repo root (no `Makefile`) | DX | No local equivalent of the (future) CI NDA scan | 10-line `Makefile` `make check` invoking the same grep list, via a shared `.ci/nda-terms.txt` | S |
| 6.2 | P3 | `list_branches` | DX | Stale merged branch `claude/populate-specs-demo-PL0Ph` | Delete; enable "auto-delete head branches" in repo settings | S |
| 7.1 | P2 | `.github/pull_request_template.md` absent | Config | No PR template to prompt NDA-check for external contributors | 6-line template with checkbox: "I ran `make check` / no internal names from `.ci/nda-terms.txt`" | S |

---

## Proof for every P0 / P1

### P0 — Finding 2.1 (missing NDA scan)

Upstream `caskey-coding/caskeycoding-specs/.github/workflows/pr-validation.yml:83–109` runs a ~20-line blocklist scan (internal tool names, attribution patterns, 12-digit AWS-account-ID regex). This block is the *only* automated gate between curator's editor and a public Git push. The demo's `.github/` directory does not exist.

**Failure mode:** the "milestone refresh" process (README:79–82) involves a human pasting content from a private repo with 138 NDA-bound spec files into this public one. Without the scan, the only backstop is the curator remembering all 14 terms. Result on failure: one leaked term on a public repo linked from a personal-brand blog inverts the repo's explicit positioning.

### P1 — Finding 2.2 (branch protection)

`list_branches` API:

```json
[{"name":"claude/populate-specs-demo-PL0Ph","protected":false},
 {"name":"main","protected":false}]
```

**Failure mode:** a mobile-session direct push to `main` bypasses both human review and any future NDA scan. The NDA scan added via 2.1 is only enforceable if a PR is *required*. Shipping 2.1 without 2.2 is half the fix.

### P1 — Finding 2.3 (informal sync process)

`README.md:79–82` says "Not continuously synced. This repo refreshes at milestones; expect drift — it's a snapshot, not a feed." Governance lives in private ADR-006 (linked from `decision/README.md:17`) not visible to reviewers of future refresh PRs. All 20 commits landed in a single ~75-minute window on 2026-04-20 — "maintainer sits down, curates, merges" with no codified checklist.

**Failure mode:** low-frequency, high-stakes manual work with no checklist is the textbook shape for human-factors incidents.

---

## Positive observations

- **Zero `secrets.*` surface.** `search_code` for `secrets.` → 0 results. Clean cloud-auth posture matches the read-only artifact positioning.
- **Crisp scope boundaries in README.** `README.md:74–88` ("What this repo is NOT") is unusually explicit — itself a security control.
- **Explicit numbering reconciliation note.** `decision/README.md:22–36` preempts reader confusion about ADR mapping.
- **Small, focused commit history.** 20 commits with clear `type(scope): summary` messages.

---

## Cross-repo patterns

1. **Reusable `nda-content-scan` action** extracted from upstream `pr-validation.yml:58–115` to `caskey-coding/.github/.github/workflows/nda-scan.yml` as a `workflow_call` reusable, with term list as input or committed `nda-terms.txt` fetched from `caskeycoding-specs`. Both repos consume the same source of truth — closing Finding 2.3 as a side effect.
2. **Shared `spec-frontmatter-validation` reusable** — lower priority, N/A for this demo.
3. **Org-level "public-repo baseline" ruleset** — GitHub ruleset applied to every public repo: require PR, require NDA-scan check, disallow force-push, auto-delete head branches on merge. Fixes 2.2 + 6.2 here and pre-empts the same gap in every future public repo.

---

## Operational Lessons addressed

| Lesson | Applies here? | Finding |
|--------|---------------|---------|
| §2 config-pair drift | No (no code) | Skipped |
| §3 ruff pin drift | No (no Python) | Skipped |
| §4 lazy SDK imports | No (no code) | Skipped |
| §5 `cdk deploy --all` fragility | No (no infra) | Skipped |

Operational lessons are language/stack-specific and genuinely don't apply to a markdown-only demo.

---

## Definition of done checklist

- [x] Inventory table + trigger-to-deploy graph (graph is empty — stated explicitly).
- [x] All 7 dimensions scored with citations (with explicit N/A for §3, §4).
- [x] Findings table with severities and effort estimates.
- [ ] Tracking issue labeled `cicd-bar-raiser` — deferred to follow-up.
- [x] Cross-repo patterns identified (3 candidates).
- [x] Operational Lessons §2–§5 explicitly addressed (each N/A for this repo).
