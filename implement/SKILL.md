---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

Before editing anything, create and switch to a new branch `feat/<spec-name>`, with `<spec-name>` the spec's name in kebab-case. If the name isn't stated, infer it, and confirm the branch name with the user before proceeding. If already on that branch, keep using it. If you were assigned a branch or worktree (as a subagent of `implement-spec`), stay on it and skip this step.

If the work is a set of tickets under `.scratch/<feature-slug>/issues/`, work the **frontier** (see `docs/agents/issue-tracker.md`) one ticket at a time, and for each ticket:

1. Set `Status: claimed` and save, before any work. A ticket already `claimed` was left half-done by an earlier session: check the branch's commits and diff, and resume it rather than restarting.
2. Do the work (below).
3. Tick its acceptance criteria as they are met. If one can't be, leave it unticked and say why under `## Comments`.
4. Once every criterion is ticked, set `Status: resolved` and commit with the ticket's file included.

Before writing code, find the architecture pattern: the ticket's `Pattern:` line, else the one the spec recorded. If neither says, call the Skill tool with "architecture-patterns" and judge. Read that pattern's reference and follow it; "none" means no pattern applies.

Use /tdd where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once all the work is done, use /code-review to review it.

Commit your work to the current branch: call the Skill tool with "commit".
