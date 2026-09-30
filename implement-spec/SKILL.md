---
name: implement-spec
description: "Implement a whole spec by running its tickets concurrently as subagents on an integration branch."
disable-model-invocation: true
---

Implement the spec and its tickets (`.scratch/<feature-slug>/`) by orchestrating subagents. You coordinate; the subagents write the code. Treat the tickets as a task graph, not a queue: the `Blocked by:` lines are its edges.

Communicate sparsely. Pass subagents pointers (spec path, ticket path, commit hashes), not restated content.

## Process

1. **Read the graph.** Read `spec.md` and every ticket under `issues/`. Compute the **frontier**: open tickets whose blockers are all closed (see `docs/agents/issue-tracker.md`).
2. **Create the integration branch** `feat/<feature-slug>` from the current branch, unless it exists. If the feature slug is unclear, confirm it with the user first.
3. **Launch the frontier.** For each frontier ticket, set `Status: claimed`, then start one **implementer subagent** in the background, in its own git worktree on branch `feat/<feature-slug>/<NN>`. Each implementer:
   - confirms its worktree is based on the integration branch before starting, and resets onto it if not;
   - calls the Skill tool with "implement" for its single ticket, staying on its assigned branch, and stops after that ticket, leaving `Status: resolved` and its ticks in the ticket file as part of its own commit;
   - merges the integration branch tip into its own branch before reporting done.
4. **Check for hidden conflicts.** If two frontier tickets would obviously touch the same files, run them one after the other instead. The graph doesn't know about file overlap; you do.
5. **Merge as tickets finish.** Hand each finished branch to a **merger subagent**: it merges into the integration branch and runs the full test suite there. If green, remove the worktree and branch. If not, send the ticket back to its implementer with the failure; never leave the integration branch red.
6. **Repeat from the new frontier.** Every merged ticket can unblock others; launch them at once. A failed ticket quarantines only its own descendants: set it back to `ready-for-agent` with the reason under `## Comments`, and keep going with everything unrelated.
7. **Review once, at the end.** When no open tickets remain, call the Skill tool with "code-review" against the point the integration branch forked from, and have a single implementer subagent fix what it finds.
8. **Report** what merged, what was quarantined and why, and what the review left for the user.

State lives in the ticket files and the git branches, so a crashed session resumes from step 1: `claimed` tickets whose branch has no live worker are the ones to restart.
