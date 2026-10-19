Replace each guidance line with its content. Use the glossary's terms. Describe behaviour and modules by name; file paths and code snippets go stale.

## milestone.md

```markdown
# Milestone NNN: <Milestone title>

**Delivers:** the increment this milestone makes work, from the user's perspective.

**Done when:** a check the builder can run that proves the increment works once its last ticket lands: a test at a seam the spec names, or a command and its expected output.
```

## NNN-<ticket-slug>.md

```markdown
# Ticket NNN: <Ticket title>

**Milestone:** NNN <Milestone title>

**Status:** todo

**Depends on:** the lower-numbered tickets this one builds on directly, as `NNN, NNN`, or "None".

**Commit:** none

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective, rather than a layer-by-layer implementation list.

## Design patterns

The spec's Design Patterns rows this ticket implements, copied as they are, or "None".

## Acceptance criteria

- [ ] <behaviour>, proven at <seam>
- [ ] ...

## Notes

Left empty; the agent that builds this ticket fills it.
```
