Replace each section's guidance with its content. The first four sections are the `What` (the outcome, from the user's perspective); the next four are the `How` (the mechanism).

```markdown
# <Spec title>

**Key:** <the key the user gave, or "None">

**Depends on:** <the folder names of the specs this one is built after, such as `003-invoice-export, 005-tax-rules`, or "None">

## Problem Statement

The problem the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A long, numbered list of user stories, covering every aspect of the feature:

1. As a <actor>, I want <feature>, so that <benefit>.

For example:

1. As a mobile bank customer, I want to see the balance on my accounts, so that I can make better informed decisions about my spending.

## Out of Scope

What this spec deliberately leaves out.

## Implementation Decisions

The implementation decisions that were made, such as:

- The modules that will be built or modified, and the interfaces that change
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Describe modules and interfaces by name and behaviour; file paths and code snippets go stale.

## Relies On

For each dependency, the assumptions this spec makes about its planned state, one bullet each: an interface, a seam, a behaviour, a term. `/build` checks each one against the landed code before it builds. Without dependencies, write "None".

- `<dependency folder>`: <assumption>

## Testing Decisions

- The seams the user confirmed, each with what it catches.
- What makes a good test here: external behaviour through the seam, never implementation details.
- Prior art: similar tests already in the codebase.

## Design Patterns

One row per pattern the user confirmed, as the `design-patterns` skill records them. When none is needed, write "None needed."

| Pattern | Where | Why |
|---|---|---|

## Further Notes

Anything else about the feature, including the decisions the conversation left open, as open questions.
```
