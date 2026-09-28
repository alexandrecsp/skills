---
name: to-diagram
description: Builds a Mermaid diagram — sequence (dual business/technical layers), C4 (system/container/component architecture), flowchart (decision or process logic), or roadmap (timeline/phases). Identifies which type fits the request, confirms with the user, then builds it from real names only. Use when the user asks for a diagram, wants to visualize an architecture, a flow, a decision process, or a roadmap.
---

# To Diagram

## Real names only

A diagram with invented class, system, or component names is worse than no diagram — it documents something that doesn't exist and will mislead the next person who reads it. If any name isn't confirmed, ask a targeted question or look at the codebase yourself before generating anything; don't fill the gap with a plausible-sounding guess.

## 1. Identify the type

| Type | Use when | Template |
|---|---|---|
| Sequence | Showing interactions/calls between distinct participants over time, across boundaries | [references/SEQUENCE.md](./references/SEQUENCE.md) |
| C4 | Showing system architecture at context, container, or component level | [references/C4.md](./references/C4.md) |
| Flowchart | Showing decision logic or a process/pipeline, no distinct actors messaging each other | [references/FLOWCHART.md](./references/FLOWCHART.md) |
| Roadmap | Showing a timeline, phases, or planned milestones | [references/ROADMAP.md](./references/ROADMAP.md) |

Infer the type from what's being described (an interaction between things → sequence; "what does this system look like" → C4; "what path does this decision take" → flowchart; "what's coming and when" → roadmap). If the request genuinely fits more than one, or names none of these directly, ask rather than guessing.

## 2. Confirm before building

State which type you inferred, in one line, before producing anything. Building the wrong type wastes the user's time reviewing a diagram that doesn't answer their actual question — a one-line confirmation is cheap next to that. Skip this only when the user already named the type explicitly (e.g. "give me a sequence diagram for X").

## 3. Build from the matching template

Read the template file for the confirmed type — it carries that type's required input, structural rules, Mermaid syntax, examples, and its own verification checklist. Each template is self-contained; don't read the others.

## Output format (all types)

- An H2 title naming what the diagram covers.
- The ```mermaid code block.
- Any legend the specific template calls for (a participant list, a name mapping) below it.
