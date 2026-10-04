# Observer

## Overview

The **publisher** notifies every registered subscriber through one interface (`update()`), so it is coupled to the interface, never to concrete subscribers. The payoff is a publisher that stays unchanged while the set of listeners changes at runtime.

## When to Use

- A state change in one object must update others, and the set of dependents is unknown up front or changes at runtime
- Wiring custom reactions to UI or gameplay events (a click, a health change, a level completed)
- Subscriptions are temporary or conditional (listen only while a screen is open)
- Polling would waste work, and broadcasting to everyone would bother the uninterested

## When Not to Use

- One known dependent that always reacts belongs behind a direct call
- Use Observer when the publisher should not know, or should not be edited for, its listeners; a one-subscriber publisher needs a second in view

## Core Principles

### 1. The publisher owns the subscriber list and nothing else about them

Publisher exposes `subscribe` / `unsubscribe` and notifies through the subscriber interface. It holds no knowledge of what a subscriber does with the notification.

```typescript
interface Subscriber<E> {
  update(event: E): void;
}

class Publisher<E> {
  private subscribers = new Set<Subscriber<E>>();
  subscribe(s: Subscriber<E>) { this.subscribers.add(s); }
  unsubscribe(s: Subscriber<E>) { this.subscribers.delete(s); }
  protected notify(event: E) { for (const s of [...this.subscribers]) s.update(event); }
}
```

Share this machinery through a base class or a composed helper so each concrete publisher only decides *when* to notify.

### 2. Notify with enough context that subscribers need not reach back

Pass the event data (what changed, new value) in `update()`, so subscribers neither query the publisher nor know its concrete type. The payload carries facts in the publisher's own language (`OrderPlaced`, `HealthChanged`).

### 3. Every subscribe has an unsubscribe

A subscriber with a shorter life than its publisher unsubscribes when it dies (screen closed, object destroyed, test finished). A dangling subscription keeps the subscriber alive and runs against dead state. In Unity, register in `OnEnable`, unregister in `OnDisable`.

### 4. Assume no notification order

Subscribers run in no guaranteed order; none may depend on another having run first. If ordering matters, the dependency is real: model it explicitly (one subscriber that triggers the next, or a direct sequence).

### 5. Guard the notification loop

Iterate over a copy (or reverse index) so a subscriber that unsubscribes itself during `update()` does not corrupt the loop. Set the publisher's policy for a throwing subscriber once, so one failure does not silently starve the rest.

## Verification

- [ ] The publisher is coupled to the subscriber interface only
- [ ] Event payloads carry what subscribers need; none downcasts or queries the publisher
- [ ] Every subscription has a teardown path that unsubscribes
- [ ] No subscriber relies on notification order
- [ ] The notify loop tolerates unsubscribing during notification and defines its failure policy
