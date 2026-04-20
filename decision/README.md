# decision/

Org-wide ADRs reproduced in this demo.

- [ADR-001: Spec-Driven Development](./ADR-001-spec-driven-development.md)
- [ADR-002: Canonical SDD File Structure](./ADR-002-sdd-file-structure.md)

## About the numbering

In the **private `caskeycoding-specs` repo**, these two ADRs are
numbered `ADR-003` and `ADR-004` — they were the 3rd and 4th
org-wide ADRs written there. In the private repo:

- `ADR-001` covers spec package shapes (integration vs. domain).
- `ADR-002` covers the two-domain brand split between
  ericcaskey.com and caskeycoding.com.
- `ADR-005` was a draft that `ADR-006` superseded (see
  [ADR-006 in the private repo](https://github.com/Caskey-Coding/caskeycoding-specs/blob/master/decision/ADR-006-public-specs-demo-package.md)).
- `ADR-006` is the decision that produced _this_ demo repo.

Per ADR-006, this demo reproduces only the two ADRs that teach SDD
as a methodology. The rest are caskeycoding.com-platform-specific and
stay private. Inside this demo they are renumbered `001` and `002`
so new readers do not trip over the missing 1 and 2.

## A note on cross-references

[ADR-002 (canonical SDD file structure)](./ADR-002-sdd-file-structure.md)
is reproduced verbatim from the private repo's ADR-004, with one
exception: its `Depends on` header has been adjusted to name the
demo's `ADR-001` rather than the private repo's `ADR-003`. The body
text still describes the private repo's canonical tree and references
private-repo ADRs (the ones that live only there) by name.
