---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

Before writing code, find the architecture pattern: the ticket's `Pattern:` line, else the one the spec recorded. If neither says, call the Skill tool with "architecture-patterns" and judge. Read that pattern's reference and follow it; "none" means no pattern applies.

Use /tdd where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, use /code-review to review the work.

Commit your work to the current branch: call the Skill tool with "commit".
