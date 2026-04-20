# Feature 001 — Decision Records

> Package-scoped ADRs for the broken-link-detection feature. Numbered
> within this file. Global architectural decisions go in the repo's
> top-level `decision/` directory; this file is for decisions that are
> only meaningful within this feature.

---

## ADR-1: Use GitHub's heading-slug algorithm for anchor resolution

**Status:** `accepted`
**Date:** 2026-04-02

### Context

Internal links can target a heading anchor: `[see below](#api-interface)`.
To verify the anchor exists, we need to compute the slug for every
heading in the target file and check for a match. The question is: whose
slug algorithm?

Options surveyed:

- **goldmark default** — what the parser produces out of the box.
- **GitHub Flavored Markdown** — what readers actually see on github.com
  and what most contributors write anchors against.
- **GitLab** — different from GitHub in edge cases (emoji, repeated
  headings).
- **CommonMark spec** — unspecified; leaves it to implementations.

### Decision

Implement GitHub's algorithm. Ship it as a small dedicated package
(`internal/slug`) so it can be swapped later if the project ever adds a
GitLab or self-hosted-Gitea target.

### Rationale

The overwhelming majority of users write markdown intended to render on
GitHub. A slug algorithm that disagrees with GitHub's will produce false
positives (link looks fine on github.com, tool says broken) which is the
single worst failure mode for a CI tool — it destroys trust.

### Consequences

- Pin the algorithm to the version GitHub documents as of 2026-04. If
  GitHub changes it, we update and cut a minor release.
- Repeated-heading disambiguation (`title`, `title-1`, `title-2`) has
  dedicated test cases.
- Emoji handling: strip before slugging, same as GitHub.

### Alternatives rejected

| Alternative | Reason rejected |
|-------------|-----------------|
| goldmark default | Disagrees with GitHub on repeated headings |
| Make it configurable at v1 | Premature flexibility; pick one, add a flag only if a real user asks |
| Skip anchor checking | Internal-link checking without anchors is half the feature |

---

## ADR-2: HEAD requests, fall back to GET on 405

**Status:** `accepted`
**Date:** 2026-04-10

### Context

External link checking is dominated by network wait time. HEAD requests
are cheaper than GET — no body transferred. But some servers (notably
older nginx configs and some CDNs) return 405 Method Not Allowed on
HEAD, even for URLs that are perfectly reachable via GET.

### Decision

Issue HEAD first. On a 405 response specifically, retry with GET and
read-and-discard the body. Any other non-2xx/3xx response is treated
per the classification rules in `spec.md §Requirements`.

### Rationale

Real-world link corpora have a non-trivial tail of 405-on-HEAD sites.
Reporting them as `broken` is wrong. Always using GET would double the
bandwidth and the wall-clock time of the typical audit, for no benefit
on the 95%+ of sites that handle HEAD correctly.

### Consequences

- One extra round trip for the affected tail of URLs.
- Memory ceiling: cap read body to 1 MiB then cancel the request. We
  only need the status code, not the content.
- Test fixture server in `test/integration/fixtures/` includes a
  405-on-HEAD endpoint to lock this behavior in.

### Alternatives rejected

| Alternative | Reason rejected |
|-------------|-----------------|
| Always GET | Wasteful on the common case |
| Always HEAD | Wrong on the tail — false positives |
| Make it per-URL configurable | Users shouldn't need to know which sites are broken |
