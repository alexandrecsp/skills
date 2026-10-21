---
name: build
description: Build the next milestone of a spec under .sdd/specs/, each ticket test-first in its own sub-agent, then one review of the milestone's changes.
argument-hint: "[spec-folder or milestone-folder]"
disable-model-invocation: true
metadata:
  type: process
---

Build one **milestone** per call, from the files `to-tickets` wrote under `.sdd/specs/NNN-<slug>/`. Each ticket is built by an **implementer sub-agent** in a fresh context, the milestone closes with one `review-changes` over its whole diff, and it lands as one squash-merged PR from its own branch.

Run every implementer sub-agent on Sonnet: building a ticket test-first does not need the session's model, and the implementers spend most of the milestone's tokens. Talk to sub-agents through **context pointers**: the paths of the ticket, its `milestone.md` and the `spec.md`, rather than their contents.

## Process

1. **Pick the milestone.** With a milestone folder, take it. With a spec folder, or without an argument the spec folder from this conversation, take the lowest-numbered milestone that holds a ticket whose **Status** is not `done`. Milestones run in order: when an earlier milestone still holds such a ticket, report it and stop; when every ticket is `done`, report that the spec is built and stop. Done when you can name the milestone and its tickets that are not `done`.

2. **Check the previous milestone is merged.** Each milestone lands as one squash-merged PR, and the next one branches from the base after it. Skip this for the first milestone. Otherwise read the previous milestone's **PR** line: when it names a PR, check `gh pr view <pr> --json state` reports `MERGED`; when it is `none` or `gh` is unavailable, ask the user whether it is merged. When it is not, report it and stop. Done when the previous milestone is merged, or this is the first.

3. **Prepare the branch.** Find the **base branch**: the remote's default (`git symbolic-ref --short refs/remotes/origin/HEAD`, without `origin/`), or, without a remote, ask the user. When `git status --porcelain` is not empty, report it and stop: the commits must hold only this milestone's work. The milestone's branch is `sdd/<spec-folder>/<milestone-folder>`, such as `sdd/003-invoice-export/001-csv-writer`: the `sdd/` prefix groups the pipeline's branches, so they are listed and cleaned up with one glob. When it exists, check it out; otherwise `git fetch` and create it from `origin/<base>` (from the local base without a remote). The **fixed point**, the commit the review diffs against, is `git merge-base <base-ref> HEAD`, which survives rebases. Done when the branch is checked out and the fixed point resolves.

4. **Build the tickets.** Take the tickets that are not `done` in number order: each builds only on the tickets before it. For each, set its **Status** to `doing`, then dispatch one implementer sub-agent, in the foreground, with the paths of the ticket, `milestone.md` and `spec.md`, and this brief:

   > Build this ticket. First read the earlier tickets in its milestone folder and the tickets its **Depends on** names, their Notes, and the commits their **Commit** lines name; when such a commit is gone, its milestone was squash-merged, so read that milestone's **PR** diff (`gh pr diff <pr>`) instead. Invoke the `tdd` skill, testing at the seams the ticket's acceptance criteria name; when a criterion names no seam, report that instead of choosing one. Build its Design patterns rows at their Where. Place and name every new file by `docs/STRUCTURE.md` when it exists. Run the typechecker and the test files you touch as you go, and the full suite before you finish. Then fill the ticket's Notes with what its diff can't show the next ticket (decisions the ticket left open, departures from it, gotchas you hit) in a few bullets, or "None". Commit everything by invoking the `commit` skill with the spec folder. Then write the commit's SHA into the ticket's **Commit** line and set its **Status** to `done`. Report the commit, and anything you could not build.

   When a sub-agent could not build its ticket, set its **Status** to `blocked`, report that to the user and stop: the tickets after it may depend on it. Done when every ticket in the milestone is `done` and committed.

5. **Check the increment.** Run the full suite and the check in `milestone.md`'s **Done when**. When either fails, dispatch an implementer sub-agent with the failure, then run both again. After two failed fixes, report the failure to the user and stop. Done when both pass.

6. **Review.** Invoke the `review-changes` skill with the fixed point and the milestone folder, and show the user the report. When it flags findings **escalate**, ask the user how to resolve each one: fix it as proposed, fix it another way, or skip it. Otherwise go on without asking. Done when the user has answered every escalated finding, or none was flagged.

7. **Fix.** Fix every finding not flagged escalate, plus the escalated ones as the user resolved them, in one implementer sub-agent that runs the full suite and commits by invoking the `commit` skill with the spec folder. Do not run `review-changes` again: the sub-agent checks only the findings it fixed, so the fix does not turn into a review loop. Done when the fixes are committed with the suite passing, or there was nothing to fix.

8. **Open the PR.** Ask the user before pushing. Push the branch (`git push -u origin <branch>`) and open the PR against the base branch with `gh pr create`. The squash merge turns the PR's title and body into the one commit that lands, so write them in the `commit` skill's message format: the title is the spec's key and the milestone's increment in the present tense, the body one to three bullets. Write the PR's URL into `milestone.md`'s **PR** line. When the user declines, or there is no remote or `gh`, leave **PR** as `none`. Done when the PR is open and recorded, or the user declined.

9. **Hand over.** Report the tickets built with their commits, what the review fixed, the PR, and the next step: squash-merge the PR, then run `/build <spec-folder>` again for the next milestone; or that the spec is built. Done when the user has the report.
