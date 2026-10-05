---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

## Spec branch

All implementation of a spec lands on its **spec branch**, `spec/<spec-id>-<slug>`: the id and a short slug of the published spec. The name is the only record, so every stage finds it with `git branch --list 'spec/<spec-id>-*'`. Check it out, creating it from the default branch if absent. Work with no spec stays on the current branch; called by `implement-spec`, stay on the ticket branch it gave you.

## Build

Before writing code, settle the patterns. If a spec or ticket records a verdict (a ticket's `Pattern` field, or the spec's Implementation Decisions), follow the matching `patterns` reference; a recorded `none` means no pattern applies. If there is no spec or no record, judge afresh through the `patterns` skill from what you are building and the files it touches.

Call the Skill tool with `tdd` to build, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Commit your work to the branch you are on with /commit.

## Close

Called alone, run /code-review (it takes the spec branch's merge-base with the default branch as its fixed point), then resolve the ticket the way the issue tracker closes work. Called by `implement-spec`, report done and leave review and closing to it.

Next: `/implement-spec <spec ref>` for the remaining tickets, or `/pr` when the spec is done.
