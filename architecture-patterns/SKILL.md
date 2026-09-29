---
name: architecture-patterns
description: Index of house architecture patterns (frontend UI, API/interface design, hexagonal architecture, Unity). Use when writing a feature's design.md Approach section during /spec, or when implementing a step from its issues/<id>-<slug>.md during /implement, and you need to know which house pattern applies.
---

# Architecture Patterns

Router only. Each approach lives in its own file under `references/`, self-contained. Read the one that matches what you're about to build — don't read the others.

| Approach | File | Applies when |
|---|---|---|
| Frontend UI | [references/FRONTEND-DESIGN.md](./references/FRONTEND-DESIGN.md) | Building or modifying user-facing interfaces, components, layouts, accessibility, client-side state |
| API & interface design | [references/API-DESIGN.md](./references/API-DESIGN.md) | Designing REST/GraphQL endpoints, module boundaries, public interfaces, idempotency |
| Hexagonal architecture | [references/HEXAGONAL-PATTERN.md](./references/HEXAGONAL-PATTERN.md) | Structuring a service/feature into domain/application/ports/adapters, separating business rules from infrastructure |
| Unity | [references/UNITY.md](./references/UNITY.md) | Writing or reviewing Unity C#, gameplay systems, prefabs/ScriptableObjects, frame budget/performance |

Infer which applies from the step's description and the files it touches — a step's `issues/<id>-<slug>.md` doesn't tag it with an approach.

If a step needs an approach not listed here, use judgment and, once the approach proves itself on real work, add a new file under `references/` and a row above.
