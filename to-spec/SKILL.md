---
name: to-spec
description: "Turn the current conversation into a spec, then break it into tracer-bullet tickets, all written to the local issue tracker. Synthesis, not interview: the only question is the one checkpoint on seams and pattern."
disable-model-invocation: true
---

# To Spec

Two phases, each ending on files written to disk: a **spec**, then **tickets** cut from it. Synthesise what the conversation and codebase already tell you; the only question put to the user is the checkpoint on seams and pattern. Once approved, spec and tickets are written without further approval.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run `/setup`.

If the user passes a path to an existing `spec.md`, read it and start at phase 2.

## Phase 1: Spec

Done when `.scratch/<feature-slug>/spec.md` is written and read back complete.

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout, and respect any ADRs in the area you're touching.

2. Sketch the seams at which you'll test the feature. Existing seams beat new ones. Use the highest seam possible; if a new one is needed, propose it at the highest point you can. Fewer seams is better; the ideal is one.

3. **Judge the architecture pattern.** Call the Skill tool with "architecture-patterns", decide which house pattern(s) the work needs (or none), and read the matching reference before drafting.

4. **Checkpoint: seams and pattern.** Show the user the seams and your pattern pick with its one-line reason. Iterate until the user approves; this approval covers the spec and the tickets cut from it.

5. Draft the spec from the template in [SPEC-TEMPLATE.md](references/SPEC-TEMPLATE.md) and write it to `.scratch/<feature-slug>/spec.md`, opening with `Status: ready-for-agent`: no further triage.

## Phase 2: Tickets

Work from the spec on disk. Done when every ticket is written as its own file.

### 1. Draft vertical slices

Look for opportunities to prefactor the code to make the implementation easier: "Make the change easy, then make the easy change."

Break the work into **tracer bullet** tickets.

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests): vertical, NOT a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window
- Any prefactoring should be done first

</vertical-slice-rules>

Give each slice its **pattern**: the architecture pattern(s) its work follows, usually the spec's, but a slice can differ (a UI slice and an API slice of the same feature), or `None`.

Give each ticket its **blocking edges**: the other tickets that must complete before it can start. A ticket with no blockers can start immediately.

**Wide refactors are the exception to vertical slicing.** A **wide refactor** is one mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase, so a single edit breaks thousands of call sites at once and no vertical slice can land green. Sequence it as **expand–contract**. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites over in batches sized by blast radius (per package, per directory), each batch its own ticket blocked by the expand, keeping CI green batch to batch because the old form still exists. Finally contract: delete the old form once no caller remains, in a ticket blocked by every migrate batch. When even the batches can't stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify ticket; green is promised only there.

### 2. Audit the breakdown

Before writing, check every ticket: its size fits one fresh context window, each blocker genuinely gates it (no edge without a real dependency), and no two tickets should be merged or split further.

### 3. Write the tickets

Write the tickets to the local tracker `/setup` configured (`docs/agents/issue-tracker.md` defines the header fields):

- One file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first), never a single combined file.
- `Blocked by:` lists the numbers of the tickets that gate it, or `None`.
- `Pattern:` names the pattern(s) (keys from the `architecture-patterns` table, comma-separated), or `None`.
- `Status: ready-for-agent`, since tickets are agent-grabbable by construction.

<ticket-template>

# <NN>: <Ticket title>

Status: ready-for-agent
Blocked by: None
Pattern: hexagonal

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective, not a layer-by-layer implementation list.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

</ticket-template>

## Both phases

Keep specific file paths and code snippets out of spec and tickets: they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo.
