---
name: to-tickets
description: Break a spec, a plan, or the current conversation into milestones of tracer-bullet tickets, saved in the spec's folder under .sdd/specs/.
argument-hint: "[spec-id]"
disable-model-invocation: true
metadata:
  type: process
---

Break the work into **milestones**, each a group of **tickets**. A milestone is an increment that leaves the code working and tested once its last ticket lands. Milestones run in order: a milestone starts only when the one before it is done. A ticket is a **tracer bullet**: a narrow but complete slice that one agent implements in a fresh context. Tickets are built one at a time in number order, so each ticket builds only on tickets numbered before it, and its **Depends on** line names exactly which ones.

This needs `.sdd/settings.json`; when it is missing, tell the user to run `/setup-repository` and stop.

## Process

1. **Gather context.** With a `[spec-id]` argument, find its spec folder by [SPEC-ID.md](../to-spec/SPEC-ID.md) and read its `spec.md`; without one, use the spec written in this conversation or, failing that, the conversation itself (then each behaviour discussed stands in for a user story, and you propose the seams for the user to confirm in step 4). When the spec folder already holds a `milestones/` folder, report that and stop, unless the user asked for a rewrite. Read the code the work touches, plus the [domain docs](../domain-modeling/DOMAIN-DOCS.md). Look for **prefactoring**: a change that makes the later tickets easy. When the spec's Further Notes leave a question open that a ticket would build on, ask the user to settle it, and write the answer into Further Notes. Without a spec, follow [IN-FLIGHT.md](../to-spec/IN-FLIGHT.md) for the modules the work changes, unless the conversation already ran it; when it reports an overlapping spec, recommend `/to-spec`, which records the dependency, and stop. Done when you can name the modules the work changes, and no open question a ticket builds on is left.

2. **Cut milestones.** Number them `001` upwards, in order. Each delivers one increment you can verify with a check the builder can run, and builds only on earlier milestones. Prefactoring goes in the first milestone. Done when every user story belongs to exactly one milestone.

3. **Draft tickets.** Split each milestone into tracer bullets:

   - Each ticket cuts a narrow but complete path through every layer it needs (schema, API, UI, tests): vertical, one feature thin, across all its layers.
   - Each ticket is verifiable when it lands, through a confirmed seam: one the spec names, or one the user confirms in step 4.
   - Each ticket fits one fresh context window.
   - Prefactoring tickets come first.

   A mechanical change that breaks call sites across the codebase at once cannot land as a tracer bullet; sequence it by [WIDE-REFACTOR.md](WIDE-REFACTOR.md).

   Number the tickets `001` upwards, continuing across milestones rather than restarting, in dependency order. Give each ticket its **Depends on**: the tickets it builds on directly, in any milestone. Done when every user story is covered by some ticket's acceptance criteria, every criterion names its seam, and every **Depends on** names only lower-numbered tickets.

4. **Quiz the user.** Present the breakdown grouped by milestone: each milestone's increment and its check, then its tickets in order, each with its title, what it delivers, and its **Depends on**. Without a spec, lead with the seams you proposed, each with what it catches. Ask:

   - Without a spec: are these the right seams, and, when `.sdd/settings.json` sets a `commitKeyPattern`, which key do this work's commits carry (fully matching it, or none)?
   - Does each milestone boundary mark a meaningful increment?
   - Should any ticket or milestone be merged (too fine) or split (too coarse)?
   - Do the dependencies hold: does each ticket need exactly the tickets its **Depends on** names?

   Iterate until the user approves. Done when the user approves the breakdown, seams included, and, without a spec, has given the key or none.

5. **Write the files.** Write into the spec folder, `.sdd/specs/NNN-<slug>/`; without a spec, create one with the next free `NNN` and a slug of the work's name, holding a `spec.md` with only the title and the **Key** line of [SPEC-TEMPLATE.md](../to-spec/SPEC-TEMPLATE.md). Fill the files from [TEMPLATES.md](TEMPLATES.md):

   ```
   .sdd/specs/NNN-<slug>/
   ├── spec.md
   └── milestones/
       └── NNN-<milestone-slug>/
           ├── milestone.md
           └── NNN-<ticket-slug>.md   ← one per ticket
   ```

   Done when every milestone folder has its `milestone.md` and at least one ticket, every diagram slot holds its diagram, and the user has the spec folder's path and the next step, `/build <spec-id>`, or `/build <spec-id> auto` to build every milestone in one run.
