# Domain Model

## Overview

Domain concepts (entities, value objects, the rules that govern them) are written as classes that own their data and their behaviour, in whatever architecture or framework the code lives. A rule and the state it governs stay in one place, an instance is always valid, a business failure speaks the domain's language, and variation between kinds of the same concept is expressed by polymorphism rather than by conditionals scattered across callers.

This is how domain concepts are represented in code. Terminology, glossaries and ADRs live in the `domain-modeling` skill.

## When to Use

- Creating or changing an entity, a value object, a domain error, or a rule that governs one
- Reviewing code where domain concepts are plain data shapes with their rules in free functions elsewhere
- Reviewing a conditional that switches on a type, kind, or status to decide behaviour

## When Not to Use

- Edge shapes with no behaviour: DTOs, request/response types, persistence rows, config. These stay plain `type`/`interface`
- A data-only value with zero rules. The moment a rule (validation, derived state, a computation) attaches to that shape, it moves into a class method

## Core Principles

### 1. Domain concepts are classes, not interfaces + free functions

Model entities and value objects as classes that own their data and their behaviour, not a plain `interface`/`type` paired with free-standing functions that operate on it elsewhere in the module. A `Token` owns `isExpired()` as a method; it isn't a data shape handed to a bare `isExpired(token, now)` function living nearby:

```typescript
export class Token {
  constructor(
    readonly accessToken: string,
    readonly refreshToken: string,
    readonly expiresAt: Date,
    readonly refreshExpiresAt: Date | null,
    readonly requiresManualReauth: boolean,
  ) {}

  isExpired(now = new Date()): boolean {
    return this.expiresAt <= now;
  }
}
```

This keeps a rule and the state it governs in one place, instead of scattered across a data interface and whatever functions happen to import it. Anyone changing the rule finds it where the state lives, and a class is just as easy to fake in a test.

### 2. Invariants hold at construction

The constructor (or a static factory) enforces the invariants and throws a domain error (Principle 3) if the state would be inconsistent, so an instance is always valid rather than a plain object anyone can construct in a bad shape. Fields are `readonly`; a change that must preserve an invariant goes through a method that returns a new instance or checks the invariant before mutating.

```typescript
export class Money {
  constructor(readonly amount: number, readonly currency: Currency) {
    if (amount < 0) throw new NegativeMoneyError(amount);
  }
}
```

### 3. Domain errors are transport-agnostic

An error raised by the domain represents a business failure and knows nothing about HTTP status codes, gRPC codes, or queue-retry semantics:

```typescript
export class ManualReauthRequiredError extends Error {}
```

Translation to a transport-specific shape happens exactly once, at the outermost edge that has a transport (an exception filter, an HTTP middleware, a queue handler's catch block, a UI error boundary), never inline in the domain:

```typescript
if (err instanceof ManualReauthRequiredError) {
  return res.status(401).json({ error: { code: 'MANUAL_REAUTH_REQUIRED', message: err.message } });
}
```

A domain error carrying an HTTP status can't be raised from a queue handler or CLI without dragging HTTP along, and tests start asserting on transport details instead of the business rule. Tests assert on the domain error type directly.

### 4. Polymorphism replaces conditionals over kind

When behaviour varies by the kind of a domain concept (payment method, document type, discount, account status), give each kind its own class behind one shared abstraction, so each variant carries its own rule and a new kind is a new class, not an edit to every conditional that switches on it.

```typescript
// Conditional over kind: every new kind edits every switch like this
function fee(p: Payment): Money {
  switch (p.kind) {
    case 'card': return p.amount.percent(2.9);
    case 'pix':  return Money.zero(p.amount.currency);
    case 'boleto': return Money.of(3.5, p.amount.currency);
  }
}

// Polymorphic: each kind owns its rule
export abstract class Payment {
  constructor(readonly amount: Money) {}
  abstract fee(): Money;
}
export class CardPayment extends Payment { fee() { return this.amount.percent(2.9); } }
export class PixPayment extends Payment { fee() { return Money.zero(this.amount.currency); } }
```

Rules for doing it well:

- **Smell.** The same `switch`/`if-else` over a kind, tag or enum appears in two or more places, or one conditional grows a branch for every new variant. One small conditional in one place is not a smell.
- **Interface or abstract class.** Use an interface when variants share only a contract; use an abstract class when they share state or a skeleton of behaviour too.
- **Substitutable.** Any variant works wherever the abstraction is expected, without the caller checking which one it got. A caller that asks `instanceof` to decide behaviour is the original conditional in disguise.
- **Composition before deep inheritance.** Keep hierarchies one level deep. When variants differ along two axes, or the variation is a way of doing a job swapped at runtime, compose instead: that is `strategy`, which the design-patterns index covers. When variants are created by a runtime choice, see `factory-method`.
- **Closed sets stay closed.** For a fixed, small set of kinds known to the compiler (a result that is either `Ok` or `Err`), a discriminated union with an exhaustive switch in one place is fine, and the compiler flags the missing case when a kind is added. Polymorphism earns its place when the set is open or the conditionals are many.
- **Wait for the second variant.** One implementation is a hypothetical seam, two is a real one.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "Interfaces + functions are simpler than classes, less boilerplate for domain types" | Splits data from the rules that govern it: anyone changing the rule has to know to find the matching function elsewhere. A class keeps invariant, state and behaviour together and is harder to construct into an invalid state (Principles 1, 2). |
| "Validation belongs in the controller/form, the object can just hold data" | Every other entry point (queue, CLI, test) can then build an invalid instance. The constructor is the one place all of them pass through (Principle 2). |
| "The exception can just carry an HTTP status, simpler than mapping later" | Once a domain error knows about HTTP, it can't leave HTTP (Principle 3). |
| "A switch on `kind` is easy to read" | Easy until the second and third copy of it exist; then every new kind is a hunt for every switch (Principle 4). |
| "Let's make a base class so we can add kinds later" | One variant is a hypothetical seam. Wait for the second (Principle 4). |
| "Use inheritance to share this code between variants" | Share through composition unless the variants are genuinely substitutable. Reused code is not a reason for a subtype (Principle 4). |

## Verification

After modelling or reviewing domain code:

- [ ] Entities and value objects with any behaviour (validation, derived state, computation) are classes with methods, not interfaces + free functions
- [ ] Edge shapes with no behaviour (DTOs, rows, config) remain plain types
- [ ] Constructors enforce invariants and throw domain errors; fields are `readonly` or changed only through methods that keep the invariant
- [ ] Domain errors carry no transport-specific detail; translation happens once, at the transport edge
- [ ] No kind/tag/enum `switch` is repeated across callers; each variant owns its rule, or the closed set is switched exhaustively in one place
- [ ] Behaviour a variant could own is called on the variant; callers hold no `instanceof` or kind-field checks for it
- [ ] Hierarchies are one level deep, with a second variant that justifies the abstraction
