---
name: domain-modeling
description: Build and sharpen a project's domain model. Use when discussing codebase terminology, editing a GLOSSARY.md, or recording an ADR.
metadata:
  type: process
---

# Domain Modeling

Actively build and sharpen the project's domain model as you design: challenge terms, invent edge-case scenarios, and write the glossary and decisions down the moment they crystallise.

## File structure

Most repos have a single context:

```
/
├── GLOSSARY.md
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

If a `GLOSSARY-MAP.md` exists at the root, the repo has multiple contexts: follow [CONTEXTS.md](CONTEXTS.md).

Create files lazily: only when you have something to write. If no `GLOSSARY.md` exists, create a root one when the first term is resolved. If the target `docs/adr/` doesn't exist, create it when its first ADR is needed.

## During the session

### Challenge against the glossary

When the user uses a term that conflicts with the existing language in `GLOSSARY.md`, call it out immediately. "Your glossary defines 'cancellation' as X, but you seem to mean Y. Which is it?"

### Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term. "You're saying 'account': do you mean the Customer or the User? Those are different things."

### Discuss concrete scenarios

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts.

### Cross-reference with code

When the user states how something works, check whether the code agrees. If you find a contradiction, surface it: "Your code cancels entire Orders, but you just said partial cancellation is possible. Which is right?"

### Update GLOSSARY.md inline

When a term is resolved, update `GLOSSARY.md` right there, as it happens. Use the format in [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md).

`GLOSSARY.md` holds the domain's terms and their definitions, nothing else: implementation decisions belong in ADRs or the spec.

### Offer ADRs sparingly

Only offer to create an ADR when all three are true:

1. **Hard to reverse**: the cost of changing your mind later is meaningful
2. **Surprising without context**: a future reader will wonder "why did they do it this way?"
3. **The result of a real trade-off**: there were genuine alternatives and you picked one for specific reasons

If any of the three is missing, skip the ADR. Use the format in [ADR-FORMAT.md](./ADR-FORMAT.md).

## Done

Done when every term the session settled is in its `GLOSSARY.md`, and every decision that passed the three tests has been offered as an ADR. When this skill runs beside another (such as `brainstorm`), check this before that skill's session ends.
