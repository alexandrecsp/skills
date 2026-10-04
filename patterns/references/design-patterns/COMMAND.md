# Command

## Overview

The **invoker** (button, shortcut, scheduler, queue) holds a command and runs it without knowing what it does; the **receiver** holds the actual business logic. The payoff is a trigger layer decoupled from logic, and operations that can be passed around, queued, logged, replayed, or undone.

## When to Use

- Several triggers (toolbar button, menu, shortcut) must run the same operation without duplicating it
- Operations must be queued, scheduled, retried, or sent to another process (a command is serializable data)
- Undo/redo, or an audit log of every action taken
- Objects need to be configured with an operation to run later (buttons, input bindings, ability slots)

## When Not to Use

- A trigger calling one method on one receiver needs no command
- The pattern adds an object per operation; spend it only for one of the capabilities above

## Core Principles

### 1. The command interface is minimal: `execute()`

Everything the operation needs is bound into the command (receiver reference plus parameters) at creation, so invokers call `execute()` with no arguments and stay generic.

```typescript
interface Command {
  execute(): void;
}

class CopyCommand implements Command {
  constructor(private editor: Editor) {}
  execute() { this.editor.copySelection(); }
}

class Button {
  constructor(private command: Command) {}
  click() { this.command.execute(); }
}
```

### 2. Commands delegate; receivers own the logic

A concrete command is thin: it translates the request into calls on a receiver. Business rules stay in the receiver, so the same rule is reachable without a command and never duplicated across commands. Trivial commands may hold the logic themselves.

### 3. Invokers know the interface only

An invoker stores commands behind the interface and runs them. The client (composition root) builds in this order: receivers, then commands (configured with receivers), then invokers (configured with commands).

### 4. Undo is decided per command

Add `undo()` only where reversal is required. A command that can be undone records the state it needs to reverse itself (a snapshot, or the inverse operation's inputs) at `execute()` time, and one history stack of executed commands drives undo/redo. For heavy state, pair with **Memento**.

### 5. Serializable commands carry data, not live references

To queue or send a command across a boundary, make it plain data (type plus parameters) that a handler resolves to a receiver on the other side. Live service or connection references do not survive serialization.

## Verification

- [ ] The capability that justifies commands is named: sharing across triggers, queue/schedule, undo/redo, or audit
- [ ] Invokers depend on the command interface only, with no branching on concrete command type and no parameters on `execute()`
- [ ] Each command is thin and delegates to a receiver; no business rule is duplicated across commands
- [ ] Undoable commands record the state they need at `execute()`, with history managed in one place
- [ ] Commands that cross a boundary are serializable data holding no live references
- [ ] Wiring order is receivers, commands, invokers
