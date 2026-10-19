Replace each guidance line with its content. Use the glossary's terms. Describe behaviour and modules by name; file paths and code snippets go stale. The one exception is a snippet a prototype produced that states a decision more precisely than prose (a state machine, reducer, schema, or type shape): inline it, trimmed to the decision-rich parts, and note that it came from a prototype.

## milestone.md

```markdown
# Milestone NNN: <Milestone title>

**Delivers:** the increment this milestone makes work, from the user's perspective.

**Done when:** the check that proves the increment works once its last ticket lands: a demo, or a test at a seam the spec names.
```

## NNN-<ticket-slug>.md

```markdown
# Ticket NNN: <Ticket title>

**Milestone:** NNN <Milestone title>

**Blocked by:** the number and title of each ticket in this milestone that gates this one, or "None (can start immediately)".

**Status:** todo

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective, rather than a layer-by-layer implementation list.

## Design patterns

The spec's Design Patterns rows this ticket implements, copied as they are, or "None".

## Acceptance criteria

- [ ] A behaviour a test at a named seam can prove
- [ ] ...
```

Set **Status** to `done` when the ticket lands.
