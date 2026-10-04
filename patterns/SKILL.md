---
name: patterns
description: Index of house patterns, grouped into five levels (application type, architecture, interface, domain model, class-level design patterns), and how to judge which apply. Use when writing a spec's Implementation Decisions, when splitting a spec into tickets, when implementing a ticket, or when reviewing code against the chosen pattern.
---

# Patterns

Router only. Each approach lives in its own file under `references/<level>/`, self-contained. Judge each level below, then read only the files your verdict names.

| Level | Key | Approach | File | Applies when |
|---|---|---|---|---|
| Application | `frontend` | Frontend UI | [FRONTEND-DESIGN.md](./references/application/FRONTEND-DESIGN.md) | Building or modifying user-facing interfaces, components, layouts, accessibility, client-side state |
| Application | `api` | API design | [API-DESIGN.md](./references/application/API-DESIGN.md) | Designing REST/GraphQL endpoints, error shapes, pagination, idempotency |
| Application | `unity` | Unity | [UNITY.md](./references/application/UNITY.md) | Writing or reviewing Unity C#, gameplay systems, prefabs/ScriptableObjects, frame budget/performance |
| Architecture | `hexagonal` | Hexagonal architecture | [HEXAGONAL-ARCHITECTURE.md](./references/architecture/HEXAGONAL-ARCHITECTURE.md) | Structuring a service/feature into domain/application/ports/adapters, separating business rules from infrastructure |
| Interface | `interface` | Interface design | [INTERFACE.md](./references/interface/INTERFACE.md) | A consumer boundary is created or changed: module boundary, component props, port, endpoint, public contract |
| Domain | `domain-model` | Domain model | [DOMAIN-MODEL.md](./references/domain/DOMAIN-MODEL.md) | An entity, value object, domain error or rule that governs one is created or changed; a conditional switches on a kind to decide behaviour |
| Design patterns | any key in the design-patterns index | Design patterns | [INDEX.md](./references/design-patterns/INDEX.md) | A class-level problem: swappable algorithms or a conditional over variants, requests as objects (queue, undo), notifying a dynamic set of listeners, creation that varies by subtype. The index holds the keys and how to tell the patterns apart |

## Judging which patterns apply

The pipeline judges the patterns; the user is not expected to name them. Infer each level from what is being built and the files it touches. Never force a pattern onto work it doesn't fit: a config tweak or a rename needs none, and `none` is a valid verdict.

Answer the levels in order, each with its own rule:

1. **Application: one per surface.** What kind of thing is being built? A screen plus the endpoint behind it is two surfaces, so `frontend` and `api`.
2. **Architecture: at most one per area.** How is the code inside it organised? If the repo already applies one in the area you're touching (see its `CONTEXT.md`, ADRs and existing structure), continue it unless the work is exactly the retrofit. Never introduce a second structure beside the first.
3. **Interface: answer every time.** Does this work create or change a seam a consumer depends on? Every application type and architecture makes seams (props, ports, endpoints), so this level is a lens, not a menu: the verdict states yes (`interface`) or no, with the seam named in its one-line why.
4. **Domain: answer every time the work models domain concepts.** Does this work create or change an entity, value object, domain error, or a rule that governs one? It holds in any architecture, so the verdict states yes (`domain-model`) or no, naming the concept in its one-line why. Edge shapes (DTOs, rows, config) are no.
5. **Design patterns: zero or more.** Only for a real class-level problem. They are tactical and sit inside the levels above (a strategy behind a hexagonal port, an observer inside a Unity system); they never replace them. A conditional over the kind of a domain concept is `domain-model` polymorphism; a conditional over interchangeable ways of doing one job is `strategy`.

Two checks apply at every level:

- **Proportionality.** Read the reference's own "When to Use" before committing. A pattern that costs more ceremony than the work earns (hexagonal around a thin CRUD wrapper) is a wrong pick.
- **Hard to reverse?** Adopting an application type or architecture the codebase doesn't yet use is often an ADR: offer it through `domain-modeling`.

**Record the verdict where the next stage can read it.** The spec records it in Implementation Decisions, and each ticket in its `Pattern` field (the ticket templates in `to-spec`), using the keys from the table above or from the design-patterns index, plus the one-line why. A later stage that finds the record follows it (a recorded `none` included); one that finds no record judges again from this section.

If a step needs an approach not listed here, use judgment and, once the approach proves itself on real work, add a new file under its level's folder in `references/` and a row above; its key is then valid everywhere, since no other file lists the keys. A new design pattern goes in `references/design-patterns/` instead, with its row in that index.
