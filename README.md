# caskeycoding-specs-demo

A curated, public teaching artifact that demonstrates the Spec-Driven
Development (SDD) workflow used in private development at
[caskeycoding.com](https://caskeycoding.com).

This is **not** the live specs repo. The live `caskeycoding-specs` repo
stays private — 138+ spec files, evolving daily, touching NDA-bound and
commercial work. What you see here is the _shape_ of that system with one
representative example of each archetype. Ten files, not one hundred and
thirty-eight. Readers extrapolate.

If you landed here from the blog series, this is the proof surface for
[Post 2: What a spec actually is](https://caskeycoding.com/blog/sdd-007).

---

## What you'll find

```
caskeycoding-specs-demo/
├── README.md                                   ← you are here
├── decision/
│   ├── ADR-003-spec-driven-development.md      ← why SDD at all
│   └── ADR-004-sdd-file-structure.md           ← the canonical tree
├── integration-package-example/                ← fictional: link-audit
│   ├── CLAUDE.md                               ← routing file at code-repo root
│   ├── steering/
│   │   ├── project-overview.md
│   │   ├── conventions.md
│   │   └── tech-stack.md
│   └── feature/
│       └── 001-broken-link-detection/
│           ├── spec.md
│           └── decision-ADR.md
└── domain-package-example/                     ← fictional: bookshelf
    ├── product.md
    ├── domain.md
    ├── contracts.md
    └── delivery.md
```

---

## How to read this repo

Start with [`decision/ADR-003`](./decision/ADR-003-spec-driven-development.md).
It explains why every feature gets a spec before code. Then read
[`decision/ADR-004`](./decision/ADR-004-sdd-file-structure.md) — the
directory structure rules the live repo follows, org-wide.

Then pick one:

- **Integration package** — an evolving platform project with shifting
  requirements. [`integration-package-example/`](./integration-package-example/)
  shows the `steering / feature / decision` shape. Start at its
  [`CLAUDE.md`](./integration-package-example/CLAUDE.md).
- **Domain package** — a bounded greenfield app with a stable domain
  model. [`domain-package-example/`](./domain-package-example/) shows the
  `product / domain / contracts / delivery` shape. Start at
  [`product.md`](./domain-package-example/product.md).

The two examples are **fictional** — `link-audit` and `bookshelf` don't
exist. They're pedagogical sketches, written for clarity. The live repo
uses exactly these shapes for real projects; those live projects stay
private.

---

## What this repo is NOT

- **Not a mirror.** The live `caskeycoding-specs` repo is private and
  stays that way. Nothing here was extracted from it mechanically.
- **Not continuously synced.** This repo refreshes at milestones
  (material changes to ADR-003 or ADR-004; new package shape). Expect
  drift — it's a snapshot, not a feed.
- **Not part of any AI workflow.** No Claude Code session, Perplexity
  connector, or routing table points here. It's a read-only artifact for
  humans arriving from the blog.
- **Not a tutorial.** The blog series is the narrative. This repo is the
  receipts.
- **Not exhaustive.** One example per archetype. The live repo has
  dozens. The shape is the point.

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
adapt, teach with them freely. ADR-003 and ADR-004 are reproduced
verbatim from the private repo for reference; same terms.

---

## Contact

[caskeycoding.com](https://caskeycoding.com) — engineering proof-of-work.
