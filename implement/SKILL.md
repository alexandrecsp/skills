---
name: implement
description: "Implement a piece of work based on a spec or set of tickets. Pass --parallel to run the spec's tickets concurrently as subagents."
disable-model-invocation: true
---

With `--parallel`, read [PARALLEL.md](references/PARALLEL.md) and follow it instead of the rest of this file.

Implement the work described by the user in the spec or tickets.

Before editing anything, create and switch to a new branch `feat/<spec-name>`, with `<spec-name>` the spec's name in kebab-case. If the name isn't stated, infer it, and confirm the branch name with the user before proceeding. If already on that branch, keep using it.

If the work is a set of tickets under `.scratch/<spec-slug>/issues/`, work the **frontier** (see `docs/agents/issue-tracker.md`) one ticket at a time, each per [TICKET.md](references/TICKET.md). Otherwise treat the work as a single ticket and follow the same file.

Once all the work is done, use /code-review to review it, then commit any fixes: call the Skill tool with "commit".
