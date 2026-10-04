---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

Before writing code, settle the patterns. If a spec or ticket records a verdict (a ticket's `Pattern` field, or the spec's Implementation Decisions), follow the matching `patterns` reference; a recorded `none` means no pattern applies. If there is no spec or no record, judge afresh through the `patterns` skill from what you are building and the files it touches.

Use /tdd where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, use /code-review to review the work.

Commit your work to the current branch.
