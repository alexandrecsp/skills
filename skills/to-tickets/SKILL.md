---
name: to-tickets
description: Break a spec, a plan, or the current conversation into milestones of tracer-bullet tickets, saved in the spec's folder under .scratch/specs/.
argument-hint: "[spec-folder]"
disable-model-invocation: true
metadata:
  type: process
---

Break the work into **milestones**, each a group of **tickets**. A milestone is an increment that leaves the code working and tested once its last ticket lands. Milestones run in order: a milestone starts only when the one before it is done. A ticket is a **tracer bullet**: a narrow but complete slice that one agent implements in a fresh context. Tickets are built one at a time in number order, so the order carries the dependencies: each ticket builds only on the tickets numbered before it.

## Process

1. **Gather context.** With a `[spec-folder]` argument, read its `spec.md`; without one, use the spec written in this conversation or, failing that, the conversation itself (then each behaviour discussed stands in for a user story, and you propose the seams for the user to confirm in step 4). When the spec folder already holds a `milestones/` folder, report that and stop, unless the user asked for a rewrite. Read the code the work touches, plus the **domain docs** when present: `GLOSSARY.md` and the ADRs in `docs/adr/`; when a `GLOSSARY-MAP.md` exists, the root ADRs plus the `GLOSSARY.md` and `docs/adr/` of each context the work touches. Look for **prefactoring**: a change that makes the later tickets easy. Done when you can name the modules the work changes.

2. **Cut milestones.** Number them `001` upwards, in order. Each delivers one increment you can verify with a check the builder can run, and builds only on earlier milestones. Prefactoring goes in the first milestone. Done when every user story belongs to exactly one milestone.

3. **Draft tickets.** Split each milestone into tracer bullets:

   - Each ticket cuts a narrow but complete path through every layer it needs (schema, API, UI, tests): vertical, one feature thin, across all its layers.
   - Each ticket is verifiable when it lands, through a confirmed seam: one the spec names, or one the user confirms in step 4.
   - Each ticket fits one fresh context window.
   - Prefactoring tickets come first.

   A mechanical change that breaks call sites across the codebase at once cannot land as a tracer bullet; sequence it by [WIDE-REFACTOR.md](WIDE-REFACTOR.md).

   Number the tickets `001` upwards, continuing across milestones rather than restarting, in dependency order. Done when every user story is covered by some ticket's acceptance criteria, every criterion names its seam, and no ticket depends on a higher-numbered one.

4. **Quiz the user.** Present the breakdown grouped by milestone: each milestone's increment and its check, then its tickets in order, each with its title and what it delivers. Without a spec, lead with the seams you proposed, each with what it catches. Ask:

   - Without a spec: are these the right seams?
   - Does each milestone boundary mark a meaningful increment?
   - Does the ticket granularity feel right (too coarse, too fine)?
   - Does the order hold: does each ticket need only the ones before it?
   - Should any ticket or milestone be merged or split?

   Iterate until the user approves. Done when the user approves the breakdown, seams included.

5. **Write the files.** Write into the spec folder, `.scratch/specs/NNN-<slug>/`; without a spec, create one with the next free `NNN` and a slug of the work's name. Fill the files from [TEMPLATES.md](TEMPLATES.md):

   ```
   .scratch/specs/NNN-<slug>/
   ├── spec.md
   └── milestones/
       └── NNN-<milestone-slug>/
           ├── milestone.md
           └── NNN-<ticket-slug>.md   ← one per ticket
   ```

   Done when every milestone folder has its `milestone.md` and at least one ticket, and the user has the spec folder's path and the next step, `/build <spec-folder>`.
