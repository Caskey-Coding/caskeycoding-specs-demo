# ADR-003: Spec-Driven Development as the Default Process

**Status:** `accepted`
**Date:** 2026-04-16

## Context

AI-augmented development with tools like Claude Code and Cursor works best when agents have clear, structured context to load before writing code. Without specs, agents hallucinate requirements, invent APIs that don't match the platform, and produce code that passes tests but misses the intent.

The financial-reviewer project demonstrated the value of the pattern: steering docs injected at session start, feature specs scoped to implementation units, and ADRs preventing agents from re-litigating settled decisions.

## Decision

Every feature — past and future — gets a spec in `specs/` before code is written. This is not optional for any domain (frontend, backend, infra, content).

The spec process:
1. Write or update a spec in the appropriate directory
2. Reference the spec during development for requirements and edge cases
3. After shipping, update status to `completed` and note deviations
4. For multi-week projects, use a spec package (see ADR-001)

Specs are the **source of intent**. Code is the **source of truth** for what was actually built. The "Implementation Reality" section in package READMEs bridges this gap.

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| No specs — just write code | Faster for trivial features | Agents have no context, requirements are implicit |
| Specs in code comments | Co-located with implementation | Not loadable as agent context, not indexable |
| External tool (Linear, Notion) | Rich editing, collaboration | Not in the repo, not version-controlled, agents can't read it |
| CLAUDE.md only | Simple | Doesn't scale past a handful of features |

## Consequences

### Positive
- Agents load structured context before writing code — fewer hallucinated requirements
- Specs survive across conversations — new agents pick up where the last left off
- Implementation Reality tracking catches spec-code drift before it compounds
- Version-controlled alongside the codebase

### Negative
- Spec maintenance overhead — specs can go stale if not updated after shipping
- Adds a step before coding — not ideal for one-line fixes

### Neutral
- The `specs/` repo is independent from the code repos — commits are separate
