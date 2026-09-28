---
name: implementation-approaches
description: Index of house implementation approaches (frontend UI, API/interface design, hexagonal architecture, Unity). Use when writing a feature's design.md Approach section during /spec, or when implementing a step from design.md during /implement, and you need to know which house approach applies.
---

# Implementation Approaches

Router only. Each approach lives in its own file under `references/`, self-contained. Read the one that matches what you're about to build — don't read the others.

| Approach | File | Applies when |
|---|---|---|
| Frontend UI | [references/frontend-design.md](./references/frontend-design.md) | Building or modifying user-facing interfaces, components, layouts, accessibility, client-side state |
| API & interface design | [references/api-design.md](./references/api-design.md) | Designing REST/GraphQL endpoints, module boundaries, public interfaces, idempotency |
| Hexagonal architecture | [references/hexagonal-pattern.md](./references/hexagonal-pattern.md) | Structuring a service/feature into domain/application/ports/adapters, separating business rules from infrastructure |
| Unity | [references/unity.md](./references/unity.md) | Writing or reviewing Unity C#, gameplay systems, prefabs/ScriptableObjects, frame budget/performance |

Infer which applies from the step's description and the files it touches — `design.md` doesn't tag steps with an approach.

If a step needs an approach not listed here, use judgment and, once the approach proves itself on real work, add a new file under `references/` and a row above.
