---
name: architecture-patterns
description: Index of house architecture patterns (frontend UI, API/interface design, hexagonal architecture, Unity) and how to judge which one applies. Use when writing a spec's Implementation Decisions, when splitting a spec into tickets, when implementing a ticket, or when reviewing code against the chosen pattern.
---

# Architecture Patterns

Router only. Each approach lives in its own file under `references/`, self-contained. Read the one that matches what you're about to build — don't read the others.

| Key | Approach | File | Applies when |
|---|---|---|---|
| `frontend` | Frontend UI | [FRONTEND-DESIGN](./references/FRONTEND-DESIGN.md) | Building or modifying user-facing interfaces, components, layouts, accessibility, client-side state |
| `api` | API & interface design | [API-DESIGN](./references/API-DESIGN.md) | Designing REST/GraphQL endpoints, module boundaries, public interfaces, idempotency |
| `hexagonal` | Hexagonal architecture | [HEXAGONAL-PATTERN](./references/HEXAGONAL-PATTERN.md) | Structuring a service/feature into domain/application/ports/adapters, separating business rules from infrastructure |
| `unity` | Unity | [UNITY](./references/UNITY.md) | Writing or reviewing Unity C#, gameplay systems, prefabs/ScriptableObjects, frame budget/performance |

## Judging which pattern applies

The pipeline judges the pattern; the user is not expected to name it. Infer it from what is being built and the files it touches, then say which you chose and why in one line.

- **Zero, one, or several.** A change can need no pattern (a config tweak, a rename), one, or a combination (a screen plus the endpoint behind it: Frontend UI and API design). Never force a pattern onto work it doesn't fit.
- **Proportionality.** Read the reference's own "When to Use" before committing. A pattern that costs more ceremony than the work earns (hexagonal around a thin CRUD wrapper) is a wrong pick. "None" is a valid answer.
- **Follow the codebase.** If the repo already applies a pattern in the area you're touching (see its `CONTEXT.md`, ADRs and existing structure), continue it unless the work is exactly the retrofit. Don't introduce a second structure beside the first.
- **Record the verdict where the next stage can read it.** The spec records it in Implementation Decisions, and each ticket in its `Pattern:` header line (`docs/agents/issue-tracker.md`), using the keys from the table above. A later stage that finds the record follows it; one that finds none judges again from this section.
- **Hard to reverse?** Adopting a pattern the codebase doesn't yet use is often an ADR: offer it through `domain-modeling`.

If a step needs an approach not listed here, use judgment and, once the approach proves itself on real work, add a new file under `references/` and a row above; its key is then valid everywhere, since no other file lists the keys.
