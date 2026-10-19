---
name: to-tickets
description: Break a spec, a plan, or the current conversation into milestones of tracer-bullet tickets, saved in the spec's folder under .scratch/specs/.
argument-hint: "[spec-folder]"
disable-model-invocation: true
---

Break the work into **milestones**, each a group of **tickets**. A milestone is an increment that leaves the code working and tested once its last ticket lands. Milestones run in order: a milestone starts only when the one before it is done. A ticket is a **tracer bullet**: a narrow but complete slice that one agent implements in a fresh context.

## Process

1. **Gather context.** With a `[spec-folder]` argument, read its `spec.md`; without one, use the spec written in this conversation or, failing that, the conversation itself (then each behaviour discussed stands in for a user story, and you name the seams). When the spec folder already holds a `milestones/` folder, report that and stop, unless the user asked for a rewrite. Read the code the work touches, plus `GLOSSARY.md` and the ADRs under `docs/adr/` when present, and look for **prefactoring**: a change that makes the later tickets easy. Done when you can name the modules the work changes.

2. **Cut milestones.** Number them `001` upwards, in order. Each delivers one increment you can demo or verify, and builds only on earlier milestones. Prefactoring goes in the first milestone. Done when every user story belongs to exactly one milestone.

3. **Draft tickets.** Split each milestone into tracer bullets:

   - Each ticket cuts a narrow but complete path through every layer it needs (schema, API, UI, tests): vertical, one feature thin, across all its layers.
   - Each ticket is verifiable when it lands, through a seam the spec names.
   - Each ticket fits one fresh context window.
   - Prefactoring tickets come first.

   A mechanical change that breaks call sites across the codebase at once cannot land as a tracer bullet; sequence it by [WIDE-REFACTOR.md](WIDE-REFACTOR.md).

   Number the tickets `001` upwards, continuing across milestones rather than restarting, in dependency order. Give each ticket its **blockers**: the tickets in its own milestone that genuinely gate it (earlier milestones are done before it starts). Done when every user story is covered by some ticket's acceptance criteria, and every blocker has a lower number than the ticket it blocks.

4. **Quiz the user.** Present the breakdown grouped by milestone: each milestone's increment, then its tickets, each with its title, blockers, and what it delivers. Ask:

   - Does each milestone boundary mark a meaningful increment?
   - Does the ticket granularity feel right (too coarse, too fine)?
   - Does each blocker genuinely gate its ticket?
   - Should any ticket or milestone be merged or split?

   Iterate until the user approves. Done when the user approves the breakdown.

5. **Write the files.** Write into the spec folder, `.scratch/specs/NNN-<slug>/`; without a spec, create one with the next free `NNN` and a slug of the work's name. Fill the files from [TEMPLATES.md](TEMPLATES.md):

   ```
   .scratch/specs/NNN-<slug>/
   ├── spec.md
   └── milestones/
       └── NNN-<milestone-slug>/
           ├── milestone.md
           └── NNN-<ticket-slug>.md   ← one per ticket
   ```

   Done when every milestone folder has its `milestone.md` and at least one ticket, and the user has the spec folder's path.
