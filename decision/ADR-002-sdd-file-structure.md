# ADR-002: Canonical SDD File Structure — Org-Wide

**Status:** `draft`
**Date:** 2026-04-19
**Author:** Eric Caskey
**Depends on:**
- [ADR-001: Spec-Driven Development](./ADR-001-spec-driven-development.md)
- Additional org-wide ADRs (spec package shapes, two-domain brand split) live only in the private repo; not reproduced in this demo.

> **Demo note.** In this public demo repository this ADR is named
> `ADR-002`. In the private `caskeycoding-specs` repo it is numbered
> `ADR-004` (it was the 4th org-wide ADR written there). Content
> below is the private ADR-004 lightly abridged per ADR-006's
> sanitization clause. The tree and cross-references in the body
> still describe the private repo's canonical structure and name
> private-repo ADRs by their private-repo numbers. See
> [decision/README.md](./README.md).

---

## Context

The specs repo has grown organically. The two-domain split (private-repo
ADR-002) was established in April 2026 and created `ericcaskey-com/`
and `caskeycoding-com/` workspaces — but the existing flat directories
(`frontend/`, `backend/`, `infra/`, `architecture/`, `case-studies/`,
`content-strategy/`) predate that split and have not been moved.
`caskeycoding-com/CONTEXT.md` explicitly notes the reorg is pending.

At the same time, the three sub-application packages have structural inconsistency:
`financial-reviewer` is a fully realized integration package (the org's reference model),
`marathon-coach` is a domain package (also deliberate per private-repo ADR-001), and both have proper
`README.md` agent entry points. However neither has an index document in `projects/` that
matches the full package shape — the `projects/001-ai-marathon-trainer.md` and
`projects/002-financial-position-reviewer.md` index stubs exist but are thin.

The `frontend-ericcaskey` code repo has a parallel gap: it gained a `specs/` directory
during April 2026 site work but has no `CLAUDE.md`, no `steering/` subdirectory, and no
agent entry point (addressed separately in `frontend-ericcaskey/specs/sdd-file-structure.md`).

This ADR documents the full org-wide target structure, the moves required to get there,
and the sequencing rules that prevent merge conflicts with in-flight work.

---

## Decision

### 1. Target structure

The optimal structure groups all specs by **domain** first, then by **concern** within
each domain. This matches how agents are routed (`CLAUDE.md` routing table) and how
humans reason about ownership.

```
caskeycoding-specs/
│
├── CLAUDE.md                              ← agent entry point (exists, maintain)
├── CONTEXT.md                             ← human process guide (exists, maintain)
├── README.md                              ← spec index (exists — update after moves)
├── _template.md                           ← single-spec skeleton (exists, no change)
│
├── _package-template/                     ← reusable integration package skeleton
│   ├── README.md
│   ├── steering/
│   ├── feature/
│   └── decision/
│
├── _shared/                               ← cross-domain contracts (both sites)
│   ├── CONTEXT.md
│   └── architecture/
│       ├── 001-two-domain-brand-architecture.md
│       └── 002-cross-domain-design-system.md
│
├── decision/                              ← org-wide ADRs (private repo numbers)
│   ├── ADR-001-spec-package-shapes.md
│   ├── ADR-002-two-domain-brand-split.md
│   ├── ADR-003-spec-driven-development.md
│   └── ADR-004-sdd-file-structure.md      ← THIS FILE (in the private repo)
│
├── ericcaskey-com/                        ← ericcaskey.com domain (exists, extend)
│   ├── CONTEXT.md
│   ├── content/
│   │   └── 001-canonical-copy.md
│   ├── discovery/
│   │   ├── 001-brand-and-design-brief.md
│   │   └── 002-design-decisions.md
│   └── frontend/
│       └── 001-site-architecture.md
│
├── caskeycoding-com/                      ← caskeycoding.com domain (MIGRATION PENDING)
│   ├── CONTEXT.md                         ← exists — note reorg pending
│   ├── architecture/                      ← MOVE from root architecture/
│   ├── frontend/                          ← MOVE from root frontend/
│   ├── backend/                           ← MOVE from root backend/
│   ├── infra/                             ← MOVE from root infra/
│   ├── case-studies/                      ← MOVE from root case-studies/
│   ├── content-strategy/                  ← MOVE from root content-strategy/
│   └── design/                            ← MOVE from root design/
│
└── projects/                              ← sub-application spec packages
    ├── 001-ai-marathon-trainer.md         ← index stub (exists, update)
    ├── 002-financial-position-reviewer.md ← index stub (exists, update)
    ├── marathon-coach/                    ← domain package (exists, no structural change)
    │   ├── README.md
    │   ├── product/
    │   ├── domain/
    │   ├── contracts/
    │   └── delivery/
    └── financial-reviewer/                ← integration package (exists, reference model)
        ├── README.md
        ├── steering/
        ├── feature/
        └── decision/
```

(Full per-file listings and moves are in the private live repo; this demo
reproduces the decision, rationale, and rules only.)

---

## What changes and what stays the same

### No structural change

| Location | Reason |
|----------|--------|
| `CLAUDE.md` | Already correct — routing table points to new target paths; update paths after moves |
| `_package-template/` | Reference skeleton — unchanged |
| `_shared/` | Already domain-correct |
| `decision/` | Org-wide ADRs live here; correct |
| `ericcaskey-com/` | Already migrated in April 2026 |
| `projects/financial-reviewer/` | Reference integration package — do not touch structure |
| `projects/marathon-coach/` | Deliberate domain package per private-repo ADR-001 — do not touch structure |

### Moves required (caskeycoding-com migration)

All moves use `git mv` to preserve history.

| From | To | Note |
|------|----|------|
| `architecture/` | `caskeycoding-com/architecture/` | 3 files |
| `frontend/` | `caskeycoding-com/frontend/` | 13 specs + assets/ |
| `backend/` | `caskeycoding-com/backend/` | 4 specs + README |
| `infra/` | `caskeycoding-com/infra/` | 10 specs + README |
| `case-studies/` | `caskeycoding-com/case-studies/` | 10 specs |
| `content-strategy/` | `caskeycoding-com/content-strategy/` | 9 files across 3 subdirs |
| `design/` | `caskeycoding-com/design/` | 1 file |

### Updates required after moves

| File | Update needed |
|------|--------------|
| `README.md` (root) | Update all directory references from `frontend/NNN` → `caskeycoding-com/frontend/NNN`, etc. |
| `CLAUDE.md` (root) | Routing table paths `frontend/` → `caskeycoding-com/frontend/`, etc. |
| `caskeycoding-com/CONTEXT.md` | Remove "Reorg status (pending)" warning block — reorg is complete |
| `projects/financial-reviewer/README.md` | Update any cross-references to old flat paths |
| `projects/marathon-coach/README.md` | Same grep |
| Internal cross-references within moved specs | grep and update to new paths |

---

## Path reference update rules

After the moves, any spec that references another spec must use paths relative to the
repo root (not relative to the file's current location). This is the existing convention
in the repo and must be preserved.

```
# BEFORE (old flat path)
../frontend/007-seo-nextjs-redesign.md

# AFTER (domain-scoped path)
caskeycoding-com/frontend/007-seo-nextjs-redesign.md
```

The `CLAUDE.md` routing table must be updated in the same PR as the moves so agents never
load a routing table that points to a directory that no longer exists.

---

## Why NOT flatten everything into `caskeycoding-com/`

The `_shared/` contracts, `decision/` ADRs, `_package-template/`, and `projects/` are
genuinely org-wide — they do not belong inside `caskeycoding-com/`. Putting them there
would force ericcaskey.com-specific specs to reach into the caskeycoding-com workspace
for shared contracts, which is the wrong dependency direction.

The two-domain split (private-repo ADR-002) established that cross-cutting concerns live
in `_shared/` and org-wide decisions live in `decision/`. This ADR preserves those rules.

---

## Spec numbering after moves

Specs keep their existing `NNN-` numbers after the move. Numbers are unique within a
directory, not across the whole repo. A `caskeycoding-com/frontend/001-minimalistic-landing.md`
and a `caskeycoding-com/case-studies/001-vitalnet-enterprise-monitoring.md` can both be
`001` — they live in different directories.

New specs added after the migration pick the next available number in their target
subdirectory.

---

## CLAUDE.md routing table — target state

After the migration, the routing table in `CLAUDE.md` becomes:

| Task | Go to | Read first |
|------|-------|------------|
| Cross-domain contract (split, DNS, analytics) | `_shared/` | `_shared/CONTEXT.md` |
| ericcaskey.com site, copy, or infra | `ericcaskey-com/` | `ericcaskey-com/CONTEXT.md` |
| caskeycoding.com site, backend, infra, or content | `caskeycoding-com/` | `caskeycoding-com/CONTEXT.md` |
| caskeycoding.com frontend spec | `caskeycoding-com/frontend/` | `caskeycoding-com/frontend/README.md` |
| caskeycoding.com backend spec | `caskeycoding-com/backend/` | `caskeycoding-com/backend/README.md` |
| caskeycoding.com infra spec | `caskeycoding-com/infra/` | `caskeycoding-com/infra/README.md` |
| caskeycoding.com case study | `caskeycoding-com/case-studies/` | — |
| Content strategy / NDA-safe messaging | `caskeycoding-com/content-strategy/` | `caskeycoding-com/content-strategy/README.md` |
| Design system | `caskeycoding-com/design/` | — |
| Project sub-app | `projects/<name>/` | `projects/<name>/README.md` |
| Platform-wide architectural decision | `decision/` | — |
| New individual spec | — | `_template.md` |
| New spec package (multi-week project) | — | `_package-template/README.md` |
| Spec index and coverage | — | `README.md` |

---

## caskeycoding-com/steering/ — decided here, not deferred

The target structure above does not include a `caskeycoding-com/steering/` directory.
This is a deliberate choice, not an oversight.

The caskeycoding.com platform already has per-area `README.md` agent instructions in
`frontend/`, `backend/`, and `infra/`. Those READMEs list tech context, spec indexes,
and numbered agent rules — they function as steering docs already. Adding a parallel
`steering/` layer on top would duplicate them without adding information.

The financial-reviewer `steering/` directory works because the project has no per-area
READMEs — `steering/` is its only always-inject surface. caskeycoding.com already has
that surface distributed across three READMEs loaded via the `CLAUDE.md` routing table.

**Decision:** no `caskeycoding-com/steering/` directory. If a platform-wide convention
is cross-cutting and not owned by frontend, backend, or infra individually, it goes into
`CLAUDE.md` §Hard Rules or a new org-level ADR in `decision/`. If the area READMEs
drift, the fix is to improve them — not to add a fourth layer.

**Shape asymmetry principle:** Integration packages (`projects/<name>/`) carry
`steering/` because they have no other always-inject surface. Domain workspaces
(`ericcaskey-com/`, `caskeycoding-com/`) route via area READMEs and `CLAUDE.md`
instead. These are two deliberate shapes, not inconsistency.

---

## Alternatives considered

| Alternative | Pros | Cons |
|-------------|------|------|
| Keep flat structure forever | Zero migration cost | Routing table is wrong; violates domain ownership from private-repo ADR-002 |
| Move everything into `caskeycoding-com/` including `_shared/` and `decision/` | Single workspace, simple | Breaks cross-domain separation |
| New repo per domain | Perfect isolation | Repo sprawl; shared ADRs need a third repo |
| Keep flat + add symlinks | No git history disruption | Symlinks in git repos are fragile; confuse grep and LLM context |

---

## Acceptance criteria

- [ ] This ADR merged to master
- [ ] All 7 root-level directories moved under `caskeycoding-com/` via `git mv`
- [ ] Root-level flat directories no longer exist
- [ ] `CLAUDE.md` routing table uses `caskeycoding-com/...` paths
- [ ] `caskeycoding-com/CONTEXT.md` §Reorg status warning block removed
- [ ] `README.md` spec index links all resolve correctly
- [ ] grep confirms no stale flat-path references
- [ ] CI/CD markdown lint passes on the PR

---

## Change log

| Date | Change |
|------|--------|
| 2026-04-19 | ADR created — initial org-wide structure analysis and migration plan |
| 2026-04-19 | Revised: decided caskeycoding-com/steering/ (no); added quiesce check; tightened gaps table |
| 2026-04-19 | Revised: renamed title to Canonical; added shape asymmetry principle |
| 2026-04-20 | Renumbered to ADR-002 for the public demo repository (private-repo number remains ADR-004) |

---

> **Note on this demo copy.** The per-file migration listings, acceptance-criteria
> grep commands, and in-flight quiesce block from the live ADR have been abbreviated
> here for readability. The decision, rationale, structure, and rules are intact.
