---
name: spec
description: Plan phase of the spec-driven workflow. Writes a feature's spec.md (what/why, Given/When/Then) and design.md (how, step checklist) into .scratch/<feature>/. Only invoke when the user explicitly runs /spec.
disable-model-invocation: true
---

# Spec

Phase 2 of the spec-driven workflow (`brainstorm` → `spec` → `implement`). Turns a converged feature scope into two frozen-enough artifacts that `implement` can execute against.

## Resolving the feature

Take the feature slug as an argument. If none is given, use the most recently modified directory under `.scratch/`. If `.scratch/<feature>/` already has a `spec.md` and/or `design.md`, read them first — you're revising, not starting fresh.

Can run without a prior `brainstorm` — a well-understood feature doesn't need the ceremony. If `docs/GLOSSARY.md` or `docs/adr/` exist, read them and stay consistent with recorded terms and decisions; if they don't exist, proceed without them.

## `spec.md` — the what/why

Stable reference of intent. Should rarely need edits once implementation starts — if it does, that's a sign the scope itself changed, not just the approach.

Structure:

```markdown
# <Feature> — Spec

## Problem
<what's broken or missing, and why it matters>

## Scope
<what's in, what's explicitly out>

## Scenarios

### 1. <scenario name>
- **Given** <precondition>
- **When** <action>
- **Then** <observable outcome>

### 2. <scenario name>
...
```

Every requirement must be expressible as a numbered Given/When/Then scenario — this is mandatory, not optional. If a requirement resists that shape, it's probably not concrete enough yet; push on it with the user before writing it down.

## `design.md` — the how

Can be revised as implementation reveals surprises; `spec.md` shouldn't need to move when this does.

Structure:

```markdown
# <Feature> — Design

## Approach
<architecture, key files/modules touched, notable trade-offs>

## Diagram
<a Mermaid diagram, only if one actually clarifies this feature — see below; otherwise `None - <reason>`>

## Contracts
<explicit contract for each shared port/interface introduced by a step — see below; otherwise `None - <reason>`>

## Steps

- [ ] **1** — <short description> (depends: [], scenarios: [1], files: [src/foo.ts])
- [ ] **2** — <short description> (depends: [1], scenarios: [2, 3], files: [src/bar.ts, src/baz.ts])
- [ ] **3** — <short description> (depends: [1], scenarios: [4], files: [src/bar.ts])
```

Before writing the Approach section, call the Skill tool with `implementation-approaches` and check whether a house approach fits (frontend, API/interface, hexagonal, Unity, or whatever's been added since) and let it shape the architecture direction. This doesn't mean tagging individual steps (still skip that, see below); it means the *Approach* paragraph itself is written with the right house convention in mind.

While settling the Approach (or anywhere else in this skill a real decision crystallizes), if it's hard to reverse, surprising without context, and the result of a genuine trade-off, invoke `domain-modeling` right there to record it as an ADR — don't wait until the whole design is done. Same for a term that turns out ambiguous or load-bearing enough to need a glossary entry. Most Approach choices won't clear that bar; when in doubt, let `domain-modeling` make the call.

Add a diagram only when a concrete trigger applies — don't leave it to a vague "would this help" judgment call:
- The feature's steps span **3 or more** collaborating files/modules/services, or
- The flow crosses a process/network boundary (client↔server, service↔external API, a queue), or
- A step's contract (see below) is easier to state as a sequence than as prose.
If none of those apply, keep the `## Diagram` heading and write `None - <reason>` rather than leaving `<...>` in the file or omitting the section — a missing section and "nothing to say here" have to read differently, or a section nobody filled in is indistinguishable from one deliberately skipped. When a diagram is warranted:
- A flow crossing system/component boundaries → invoke the `to-diagram` skill for a proper dual-layer sequence diagram.
- A structural/static view (which modules, services, or containers exist and how they relate) → a simple Mermaid flowchart or C4-style container diagram written inline, whichever fits the feature — don't force sequence diagram syntax onto a static structure question.
Use real names from the codebase either way; never invented components.

For any step that introduces or changes a **shared port, contract, or interface** other steps will depend on (e.g. a new domain entity's states, an API's error semantics, a port's method contract) — write it explicitly in the `## Contracts` section: what states/errors exist, who owns deciding what, what a caller can assume. This is exactly the class of assumption that, left implicit, turns into an expensive mid-implementation rewrite once `implement`'s final review finds the gap — write it down once during planning instead. If no step introduces a shared contract, keep the `## Contracts` heading and write `None - <reason>` rather than dropping the section — the same reasoning as `## Diagram` above.

Rules for the step list:
- Each step gets a stable numeric id, referenced by later steps' `depends`.
- `depends: [ids]` lists prerequisite steps — leave `[]` if none. This is a DAG, not a linear sequence; `implement --parallel` topologically sorts it into waves.
- `scenarios: [ids]` references the `spec.md` scenario(s) this step satisfies — `implement` uses this to write that step's tests when TDD applies, and the final spec-wide review uses the full set across all steps to check the finished feature against every scenario.
- `files: [paths]` lists the files this step actually touches. `implement --parallel` uses this to catch file-level conflicts between steps that the `depends` DAG doesn't otherwise know about (two independent steps editing the same file).
- Steps stay unchecked here — `implement` is what flips `[ ]` to `[x]` as it works, and this file is the live source of truth for progress across sessions.

Don't tag a step with an "approach" (frontend/API/hexagonal/etc.) — `implement` infers that from the step's description and the files it touches.

## Done

Report the two file paths. Don't start implementing — that's `implement`'s job.
