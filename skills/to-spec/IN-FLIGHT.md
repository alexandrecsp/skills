An **in-flight** spec is a spec under `.sdd/specs/` whose `spec.md` has neither a **Landed** nor an **Abandoned** line: drafted, ticketed, building, or built but not on the base yet. Its code is missing or half-built, so its `spec.md` is the truth about what that code will be: its **planned state**.

1. **Find the overlaps.** Dispatch one `alper-skills:scout` agent with the modules the topic touches and the paths of every in-flight `spec.md`. For each spec, it compares those modules with the modules its Implementation Decisions build or change and the Where of its Design Patterns rows, and reports the specs that share a module, naming the modules shared. Done when every in-flight spec is reported as overlapping, with its shared modules, or not.

2. **Read the planned state** of each overlapping spec: its `spec.md`, then the Notes of its `done` tickets, where its build recorded departures from the plan. Where a Note and the `spec.md` disagree, the Note wins. Done when you can state, per shared module, what the spec leaves it as.

3. **Report** to the user which specs the work builds on and the modules each shares. Design against their planned state, never the code those modules hold now.

An in-flight spec stays as written. When the work needs one to change (a different interface at a shared seam, a renamed term), ask the user whether to design around it or to revise that spec separately. When the user says an overlapping spec will never be built, write `**Abandoned:** <the user's reason>` into its `spec.md`, under its **Key** line, and drop it from the overlaps.
