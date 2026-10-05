# State

## Overview

The **context** holds one state object and delegates state-specific work to it; a transition replaces that object. The payoff is a context that stops growing with every new state, and states that change, test, and merge independently. The object appears to change its class.

## When to Use

- An object behaves differently per state (a document in draft/moderation/published, an order, a connection, a character) and the behaviour methods repeat the same state checks
- A large conditional (`if`/`switch` on a state field or enum) appears in many methods
- States multiply or their rules change often
- Similar states duplicate code that a shared base state can hold

## When Not to Use

- A machine with a few stable states and light transitions fits an enum plus a `switch`
- Behaviour that does not change with state is not a state; a client-chosen, independent algorithm is [Strategy](./STRATEGY.md)
- A state class needs a second state with different behaviour in view; one state is a field

## Core Principles

### 1. Name the states and the transitions before extracting

List every state and every legal transition (state, event, next state). Each state becomes a class; a transition nobody can name does not get a code path. If two "states" behave identically, they are one state with different data.

### 2. The state interface is the context's whole state-dependent surface

One method per operation whose behaviour varies by state, shared by every state. Behaviour that is state-independent stays in the context.

```typescript
interface DocumentState {
  publish(doc: Document): void;
  reject(doc: Document): void;
}

class Draft implements DocumentState {
  publish(doc: Document) { doc.setState(new Moderation()); }
  reject(doc: Document) { /* nothing to reject */ }
}

class Document {
  constructor(private state: DocumentState = new Draft()) {}
  setState(state: DocumentState) { this.state = state; }
  publish() { this.state.publish(this); }
  reject() { this.state.reject(this); }
}
```

### 3. The context delegates and never branches on the state

The context holds the interface, forwards each operation, and never checks which state it holds. A branch on the current state in the context means the conditional only moved.

### 4. Transitions live in the states

A concrete state decides the next state and sets it through the context's setter. States may know their successors; that coupling is the pattern, and it is what separates State from Strategy. Keep the transition table readable in one place per state; when states must not name each other, the context or a table owns transitions instead.

### 5. Each operation is defined in every state

An operation illegal in a state is an explicit decision there (ignore, throw, no-op), never an omission. Share defaults through a base state so only exceptions are written out.

### 6. State objects are stateless or own their data deliberately

A state with no fields is shared as one instance (or a static); the context's data stays in the context and arrives as an argument. A state that carries per-context data is created per transition.

## Verification

- [ ] States and legal transitions are listed, one class per behaviourally distinct state
- [ ] The context references the state interface only and never branches on which state it holds
- [ ] Every state handles every operation; illegal ones are explicit
- [ ] Transitions are set by states through the context's setter, in one place per state
- [ ] No state-enum `switch` remains in the context's methods
