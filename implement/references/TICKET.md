Work one ticket, end to end, on the branch you are on:

1. Set `Status: claimed` and save, before any work. A ticket already `claimed` was left half-done by an earlier session: check the branch's commits and diff, and resume it rather than restarting.
2. Find the architecture pattern: the ticket's `Pattern:` line, else the one the spec recorded. If neither says, call the Skill tool with "architecture-patterns" and judge. Read that pattern's reference and follow it; "none" means no pattern applies.
3. Do the work. Use /tdd where possible, at pre-agreed seams. Run typechecking regularly, single test files regularly, and the full test suite once at the end.
4. Tick its acceptance criteria as they are met. If one can't be, leave it unticked and say why under `## Comments`.
5. Once every criterion is ticked, set `Status: resolved` and commit with the ticket's file included: call the Skill tool with "commit".
