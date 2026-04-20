# caskeycoding-specs-demo

A curated, public teaching artifact that demonstrates the Spec-Driven
Development (SDD) workflow used in private development at
[caskeycoding.com](https://caskeycoding.com).

This is **not** the live specs repo. The live `caskeycoding-specs` repo
stays private — 138+ spec files, evolving daily, touching NDA-bound and
commercial work. What you see here is the _shape_ of that system with one
representative example of each archetype, written as fictional pedagogical
sketches. Twenty files, not one hundred and thirty-eight. Readers
extrapolate.

If you landed here from the blog series, this is the proof surface for
[Post 2: What a spec actually is](https://caskeycoding.com/blog/sdd-007).

---

## What you'll find

```
caskeycoding-specs-demo/
├── README.md                                                   ← you are here
├── decision/                                                   org-wide ADRs
│   ├── README.md                                               numbering notes
│   ├── ADR-001-spec-driven-development.md                      why SDD at all
│   └── ADR-002-sdd-file-structure.md                           the canonical tree
│
├── integration-package-example/                                fictional: link-audit
│   ├── README.md                                               ← package agent entry
│   ├── CLAUDE.md                                               sample code-repo routing file
│   ├── steering/                                               always-inject docs
│   │   ├── project-overview.md
│   │   ├── conventions.md
│   │   ├── tech-stack.md
│   │   └── architecture.md                                     ASCII diagrams + module tree
│   ├── feature/
│   │   ├── 001-broken-link-detection/
│   │   │   └── requirements.md                                 status: in-progress
│   │   └── 002-github-action-wrapper/
│   │       └── requirements.md                                 status: draft
│   └── decision/                                               package-scoped ADRs
│       ├── ADR-001-go-as-implementation-language.md
│       └── ADR-002-sarif-as-default-output-format.md           amends feature/001
│
└── domain-package-example/                                     fictional: bookshelf
    ├── README.md                                               ← package agent entry
    ├── product/                                                what and why
    │   ├── 001-requirements.md
    │   └── 002-use-cases.md
    ├── domain/                                                 core model + business rules
    │   ├── 001-model.md
    │   └── 002-data-contracts.md
    ├── contracts/                                              interfaces
    │   └── 001-api.md
    └── delivery/                                               build, test, ship
        ├── 001-acceptance-tests.md
        └── 002-rollout-plan.md
```

> **Numbering note.** The two ADRs in `decision/` are numbered 001 and
> 002 inside this demo. In the private `caskeycoding-specs` repo the
> same documents are numbered ADR-003 and ADR-004 — they were the 3rd
> and 4th ADRs written there. See [`decision/README.md`](./decision/README.md)
> for the full mapping.

---

## How to read this repo

**Start at the framework.** Read [`decision/ADR-001`](./decision/ADR-001-spec-driven-development.md)
for why every feature gets a spec before code. Then
[`decision/ADR-002`](./decision/ADR-002-sdd-file-structure.md) for the
canonical directory structure the live repo follows.

**Then pick an archetype:**

- **Integration package** — for projects that integrate into an evolving
  platform with shifting requirements. Read
  [`integration-package-example/README.md`](./integration-package-example/README.md)
  first — it's the agent entry point, with a Directory Map, an
  Implementation Sequence, numbered Agent Instructions, and an
  **Implementation Reality** section that demonstrates the spec-vs-code
  drift tracking pattern. Then follow the links into `steering/`,
  `feature/`, and `decision/`.
- **Domain package** — for greenfield apps with a stable bounded domain.
  Read [`domain-package-example/README.md`](./domain-package-example/README.md)
  first — it's the agent entry point, with a Spec Index tree, ASCII
  architecture and data-flow diagrams, a Key Architectural Decisions
  table, and a Module Structure. Then follow the links into `product/`,
  `domain/`, `contracts/`, and `delivery/`.

The two examples are **fictional**. `link-audit` and `bookshelf` don't
exist. They're written for clarity, modeled directly on the shapes of
real production packages in the private repo (the `link-audit` shape
mirrors `financial-reviewer`; the `bookshelf` shape mirrors
`marathon-coach`). Reading the README of either example tells you what
agents actually load when a session opens against the live equivalents.

---

## What this repo is NOT

- **Not a mirror.** The live `caskeycoding-specs` repo is private and
  stays that way. Nothing here was extracted from it mechanically.
- **Not continuously synced.** This repo refreshes at milestones
  (material changes to ADR-001 or ADR-002; new package shape). Expect
  drift — it's a snapshot, not a feed.
- **Not part of any AI workflow.** No Claude Code session, Perplexity
  connector, or routing table points here. It's a read-only artifact for
  humans arriving from the blog.
- **Not a tutorial.** The blog series is the narrative. This repo is the
  receipts.
- **Not exhaustive.** One example per archetype. The live repo has
  dozens. The shape is the point.

---

## What to look for as you read

Beyond shape, three load-bearing patterns are deliberately surfaced
across the example files:

1. **Implementation Reality.** The integration package's `README.md`
   carries an "Implementation Reality" section that tracks
   spec-but-not-in-code, dead config surfaces, and follow-up PRs. ADR-001
   calls this out as the bridge between intent (specs) and truth (code).
2. **ADRs amend each other.** `integration-package-example/decision/ADR-002`
   amends `feature/001`'s original API default. The amendment is recorded
   in both files, with the change log showing the date.
3. **Specs accumulate.** `feature/001` is `in-progress`; `feature/002`
   is `draft` and depends on `001`. The numbered subdirectories
   demonstrate that integration packages grow one feature at a time.

---

## The blog series

The 007 series on [caskeycoding.com](https://caskeycoding.com) tells the
story that surrounds these specs:

- **Post 1 — Phone-First.** Shipping production sites, a blog, and two
  AI products from a phone. The interaction model is the story.
- **Post 2 — What a spec actually is.** The engineering thesis. Links
  back to this repo as the proof surface.
- **Post 3 — The numbers.** 138 spec files. 47 spec PRs. 99 production
  code PRs across three repos. This repo is one sample of the 138.

---

## License

The sample specs in `integration-package-example/` and
`domain-package-example/` are fictional and released under CC0 — copy,
adapt, teach with them freely. ADR-001 and ADR-002 are reproduced
verbatim (ADR-002 lightly abridged) from the private repo for reference;
same terms.

---

## Contact

[caskeycoding.com](https://caskeycoding.com) — engineering proof-of-work.
