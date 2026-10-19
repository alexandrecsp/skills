---
name: build
description: Build the next milestone of a spec under .scratch/specs/, each ticket test-first in its own sub-agent, then one review of the milestone's changes.
argument-hint: "[spec-folder or milestone-folder]"
disable-model-invocation: true
metadata:
  type: process
---

Build one **milestone** per call, from the files `to-tickets` wrote under `.scratch/specs/NNN-<slug>/`. Each ticket is built by an **implementer sub-agent** in a fresh context, and the milestone closes with one `review-changes` over its whole diff.

Talk to sub-agents through **context pointers**: the paths of the ticket, its `milestone.md` and the `spec.md`, rather than their contents.

## Process

1. **Pick the milestone.** With a milestone folder, take it. With a spec folder, or without an argument the spec folder from this conversation, take the lowest-numbered milestone that holds a ticket whose **Status** is `todo`. Milestones run in order: when an earlier milestone still holds a `todo` ticket, report it and stop; when every ticket is `done`, report that the spec is built and stop. Done when you can name the milestone and its `todo` tickets.

2. **Pin the fixed point**, the commit the review diffs against: `HEAD` when none of this milestone's tickets is done yet, otherwise the parent of its first ticket commit (`git log --grep "Ticket NNN"`). Done when `git rev-parse` resolves it.

3. **Build the tickets.** Take the `todo` tickets in number order: each builds only on the tickets before it. For each, dispatch one implementer sub-agent, in the foreground, with the paths of the ticket, `milestone.md` and `spec.md`, and this brief:

   > Build this ticket. First read the earlier tickets in its milestone folder, their Notes, and their commits (`git log --grep "Ticket NNN"`). Invoke the `tdd` skill, testing at the seams the ticket's acceptance criteria name; when a criterion names no seam, report that instead of choosing one. Build its Design patterns rows at their Where. Run the typechecker and the test files you touch as you go, and the full suite before you finish. Then fill the ticket's Notes with what its diff can't show the next ticket (decisions the ticket left open, departures from it, gotchas you hit) in a few bullets, or "None". Set its **Status** to `done` and commit everything as `Ticket NNN: <Ticket title>`. Report the commit, and anything you could not build.

   When a sub-agent could not build its ticket, report that to the user and stop: the tickets after it may depend on it. Done when every ticket in the milestone is `done` and committed.

4. **Check the increment.** Run the full suite and the check in `milestone.md`'s **Done when**. When either fails, dispatch an implementer sub-agent with the failure, then run both again. After two failed fixes, report the failure to the user and stop. Done when both pass.

5. **Review.** Invoke the `review-changes` skill with the fixed point and the milestone folder, show the user the report, and ask which judgement calls to fix. Done when the user has answered, or the review found nothing to fix.

6. **Fix.** Fix every Spec finding and hard violation, plus the judgement calls the user picked, in one implementer sub-agent that runs the full suite and commits as `Milestone NNN: review fixes`. Done when the fixes are committed with the suite passing, or there was nothing to fix.

7. **Hand over.** Report the tickets built with their commits, what the review fixed, and the next milestone (run `/build <spec-folder>` again), or that the spec is built. Done when the user has the report.
