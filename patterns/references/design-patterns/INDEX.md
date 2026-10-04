# Design Patterns

Tactical patterns that usually sit inside an approach from the main router (a strategy behind a hexagonal port, an observer inside a Unity system). Read the file for the pattern you're about to apply; the others stay closed. The keys below are valid in a spec's Implementation Decisions and in a ticket's `Pattern` field.

| Key | Pattern | File | Applies when |
|---|---|---|---|
| `strategy` | Strategy | [STRATEGY.md](./STRATEGY.md) | Interchangeable algorithms behind one interface, a large conditional selecting between variants, runtime-swappable behaviour |
| `command` | Command | [COMMAND.md](./COMMAND.md) | Requests as objects: one operation from several triggers, queue/schedule/retry, undo/redo, audit log |
| `observer` | Observer | [OBSERVER.md](./OBSERVER.md) | One object's changes must notify an open or dynamic set of dependents: UI/gameplay events, temporary subscriptions |
| `factory-method` | Factory Method | [FACTORY-METHOD.md](./FACTORY-METHOD.md) | Product type unknown until runtime or subclass-chosen, creation extended without editing its users, pooled/cached creation |

## Telling them apart

Only the keys in the table are valid; State, Template Method, Decorator and Mediator appear below as contrasts, not as options.

- **Strategy vs Command**: Strategy swaps alternative ways of doing one job; Command packages one request as an object to queue, log, or undo.
- **Strategy vs State**: a strategy is chosen by the client and stays independent of the others; a state changes the context itself and states know each other.
- **Strategy vs Template Method**: Strategy varies behaviour by composition at runtime; Template Method fixes it by inheritance at class-definition time.
- **Strategy vs Decorator**: Strategy changes how an object works inside; Decorator changes what it looks like from outside.
- **Command vs Observer**: Command directs one action at one receiver; Observer broadcasts a fact to any number of listeners.
- **Observer vs Mediator**: Observer sets up dynamic one-way links; Mediator removes direct links by routing through a hub (and can be built on Observer).
- **Factory Method vs Template Method**: Factory Method is a Template Method specialised for creation, and often one step of one.

## Adding a pattern

Add a file here and a row above; its key is then valid everywhere, since no other file lists the keys.
