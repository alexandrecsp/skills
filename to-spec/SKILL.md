---
name: to-spec
description: "Turn the current conversation into a spec, then split it into tracer-bullet tickets, both published to the project issue tracker: no interview, just synthesis of what you've already discussed."
disable-model-invocation: true
---

Take the current conversation context and codebase understanding and produce a spec, then the tickets that implement it. Synthesize what you already know; the user is asked only the two checks below. If the user passes a reference (an issue number, URL or path) as an argument, fetch its full body and comments and treat it as the conversation.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run `/setup-repository`.

## Process

1. **Explore** the repo, if you haven't already, until you can name the modules the feature touches. Use the project's domain glossary vocabulary throughout, and respect any ADRs in the area. Note prefactoring opportunities for the slicing step.

2. **Judge the pattern** by calling the Skill tool with `patterns`. Continue once you hold the **verdict**: the pattern keys, or `none`, with a one-line why.

3. **Sketch the seams** at which you'll test the feature. Prefer existing seams to new ones, and the highest seam possible; the fewer seams across the codebase the better, ideally one. Propose any new seam at the highest point you can.

4. **Check the seams and the verdict** with the user. Continue once they confirm both match their expectations.

5. **Write and publish the spec** using [references/SPEC-TEMPLATE.md](references/SPEC-TEMPLATE.md), then publish it to the project issue tracker with the `ready-for-agent` triage label, no further triage. Continue once the spec has a tracker reference.

6. **Split into tickets** by reading [references/TICKETS.md](references/TICKETS.md) and following it through quiz and publish. Done when every ticket is published with its blocking edges, a `Pattern` value (a key or an explicit `none`), and the spec as its parent.

Next: `/implement-spec <spec ref>`.
