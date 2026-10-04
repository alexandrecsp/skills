# Strategy

## Overview

The **context** holds one strategy and delegates to it; the client picks which. The payoff is a context that stops growing with every new variant, and variants that change, test, and merge independently.

## When to Use

- One class carries several variants of the same job (routing by car/walk/transit, pricing rules, export formats) and each new variant grows it
- A large conditional (`if`/`switch` on a type or mode) selects between algorithm variants
- Similar classes differ only in how they perform one step
- The variant must change at runtime, or be chosen by configuration
- Business logic should stay ignorant of an algorithm's internals

## When Not to Use

- Two or three stable variants rarely earn the interface plus classes; a function parameter or a plain `switch` is the lighter answer
- In a language with first-class functions, pass the function
- A one-implementation interface needs a second variant or a test fake in view

## Core Principles

### 1. Name the axis of variation before extracting

State what varies ("how the route is computed") and what stays ("the navigator renders and tracks position"). The strategy interface covers only the varying part; the context keeps everything else. If you can't name the axis, you are splitting by accident.

### 2. The interface is the algorithm's contract, narrow and shared

One method (or a few) that every variant can honour with the same inputs and outputs. When a variant needs data the others don't, pass it at construction, not through a widened interface. A strategy never reads or mutates the context's internals; it receives what it needs as arguments.

```typescript
interface RouteStrategy {
  build(from: Point, to: Point): Route;
}

class Navigator {
  constructor(private strategy: RouteStrategy) {}
  setStrategy(strategy: RouteStrategy) { this.strategy = strategy; }
  route(from: Point, to: Point): Route { return this.strategy.build(from, to); }
}
```

### 3. The context depends on the interface only

The context never names a concrete strategy and never checks which one it holds. The client (or a composition root / DI wiring) creates the strategy and hands it in.

### 4. Selection lives in one place

Choosing the strategy (from a setting, a user choice, a request field) happens once, at the edge or in a small key-to-strategy lookup, not scattered through the context or its callers.

## Verification

- [ ] The varying axis is named; the context holds only what stays constant
- [ ] Every strategy implements the same narrow interface, with no parameter that exists for one variant only, and is substitutable for any other
- [ ] No strategy imports or mutates the context's internals
- [ ] The context references the interface, never a concrete strategy, and never branches on which one it holds
- [ ] Strategy selection happens in one place