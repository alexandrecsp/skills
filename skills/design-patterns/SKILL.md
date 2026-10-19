---
name: design-patterns
description: The catalogue of design patterns, with the rules for choosing one. Use when designing or reviewing code, or deciding whether a problem calls for a pattern.
metadata:
  type: reference
---

Each pattern name below is used exactly as written: the name alone carries its structure, so this catalogue says only when to choose each one.

## Choosing

- **Choose a pattern only for a problem the code has.** Name that problem as the pattern's "Why". Code that has no such problem gets no pattern; an unneeded pattern is Speculative Generality.
- **Prefer what the codebase already uses.** Search for the patterns already in play in the area touched, and choose the same pattern for the same problem.
- **A pattern from outside this catalogue** may be chosen when a well-known source names it. Give it the same "Why".

## Recording

Wherever a pattern choice is written down, record it as Pattern, Where and Why, one row per choice. "Where" names the module in glossary terms, not a file path.

| Pattern | Where | Why |
|---|---|---|
| Strategy | Pricing rules in the Order intake module | Three pricing schemes vary per Customer |

## Catalogue

**Creation**
- **Factory Method**: callers need an object whose concrete type depends on runtime input. Skip when one concrete type exists.
- **Abstract Factory**: families of related objects must be created together and swapped as a set (per tenant, per platform).
- **Builder**: an object has many optional parts or must be assembled in steps. Skip when a constructor or an options object reads clearly.
- **Singleton**: choose only for a resource that is genuinely one per process and cannot be injected. Prefer Dependency Injection.

**Structure**
- **Adapter**: an existing or third-party interface must fit the interface your code expects.
- **Facade**: callers need one simple entry into a tangle of collaborators. Often the shape a deepened module takes.
- **Decorator**: behaviour must be layered onto an object at runtime (caching, retries, logging) without changing its interface.
- **Composite**: a tree of parts and wholes must be treated uniformly.
- **Proxy**: access to an object must be controlled, deferred or remoted behind its own interface.

**Behaviour**
- **Strategy**: one step has interchangeable algorithms chosen per call or per configuration.
- **State**: an object's behaviour changes with an explicit state, and the legal transitions matter.
- **Observer**: several parts must react to a change without the source knowing them.
- **Command**: actions must be queued, logged, undone or retried as values.
- **Template Method**: several variants share a fixed sequence and differ in a few steps. Prefer Strategy when the steps vary independently.
- **Chain of Responsibility**: a request passes through handlers until one takes it (middleware, validation chains).

**Application**
- **Repository**: domain code reads and writes aggregates without knowing the storage.
- **Unit of Work**: several changes must commit or roll back together.
- **Dependency Injection**: a module needs collaborators that tests or configuration replace.
- **Ports and Adapters**: core logic must stay independent of transports and third parties.
- **Value Object**: a domain concept is defined by its values and needs validation in one place (Money, Email). The cure for Primitive Obsession.
- **Domain Event**: one part of the domain must react to something that happened in another without a direct call.
- **Pipeline**: data passes through an ordered series of independent transformation steps.
- **Result type**: expected failures must be returned as values the caller handles, not thrown.
