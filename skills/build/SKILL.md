---
name: build
description: Build the next milestone of a spec under .sdd/specs/, each ticket test-first in its own sub-agent, then one review of the milestone's changes.
argument-hint: "[spec-folder or milestone-folder]"
disable-model-invocation: true
metadata:
  type: process
---

Build one **milestone** per call, from the files `to-tickets` wrote under `.sdd/specs/NNN-<slug>/`. Each ticket is built by an **implementer sub-agent** in a fresh context, the milestone closes with one `review-changes` over its whole diff, and it is squashed from its own branch into the spec's branch as one commit. Once every milestone is in, the spec's branch lands on the base (see **Land the spec**). Everything stays local: never push, fetch or open a PR; the user pushes by hand.

Dispatch every implementer sub-agent as the `alper-skills:implementer` agent. Talk to sub-agents through **context pointers**: the paths of the ticket, its `milestone.md` and the `spec.md`, rather than their contents.

## Branches

The spec's branch is `sdd/<spec-folder>/spec` and each milestone's is `sdd/<spec-folder>/M<NNN>`, with `NNN` the milestone folder's number, such as `sdd/003-invoice-export/spec` and `sdd/003-invoice-export/M001`: the `sdd/<spec-folder>/` prefix groups a spec's branches, so they are listed and cleaned up with one glob. The **base branch** is the **Base** line of `spec.md`; when it has none, take the remote's default (`git symbolic-ref --short refs/remotes/origin/HEAD`, without `origin/`), or, without a remote, ask the user, and write it into a **Base** line under the **Key** line.

## Phase

Once its tickets are `done`, a milestone is still closing: reviewed, fixed, checked, squashed. Steps 5 to 8 each begin by writing their word into `milestone.md`'s **Phase** line, above **Squash**: `review`, `fix`, `check`, `squash`. Step 5 writes `ask` before asking the user about escalated findings, and step 7 writes `failed` when it gives up on a failing check: the dashboard shows both as waiting on the user.

A build that stopped **resumes** from the Phase line of the milestone step 1 picks: `none` at step 4, `review` or `ask` at step 5, `fix` at step 6, `check` or `failed` at step 7, `squash` at step 8. Steps 1 to 3 run on every call.

## Process

1. **Pick the milestone.** When `spec.md` has an **Abandoned** line, report it and stop. A milestone is **open** until its **Squash** names a commit. With a milestone folder, take it; when an earlier milestone is still open, report that and stop. With a spec folder, or without an argument the spec folder from this conversation, take the lowest-numbered open milestone; when none is open, follow **Land the spec** instead. Done when you can name the milestone and the step its Phase resumes at.

2. **Check the dependencies.** Skip this unless the picked milestone is the spec's first and none of its tickets has left `todo`: a later call already passed it. For each spec named in the **Depends on** line of `spec.md`, which is absent or `None` when there are none: when its folder is missing under `.sdd/specs/`, report it and stop; when its `spec.md` has an **Abandoned** line, report that this spec must be revised to drop it, and stop; when its `spec.md` has no **Landed** line, or `git merge-base --is-ancestor <its Landed SHA> <its Base>` fails, report that this spec waits on it and stop. Then dispatch one `alper-skills:reviewer` agent per dependency, in parallel, with the paths of both `spec.md` files and this brief:

   > Check whether the dependency was built as this spec assumes. For each bullet of this spec's **Relies On** section that names the dependency, read the code on the base and the Notes of the dependency's tickets, and report the bullet as **holds** or **drifted**, with what the code does instead. Change nothing.

   When any bullet drifted, show the user the report and stop: revising the spec is the user's call. Done when every dependency has landed and every Relies On bullet holds, or the spec has none.

3. **Prepare the branches.** When `git status --porcelain` is not empty, report it and stop: the commits must hold only this milestone's work. Find the base branch by **Branches**. When the spec's branch does not exist, create it from the base. When the milestone's branch exists, check it out; otherwise create it from the spec's branch. The **fixed point**, the commit the review diffs against, is `git merge-base sdd/<spec-folder>/spec HEAD`, which survives rebases. Done when the milestone's branch is checked out and the fixed point resolves.

4. **Build the tickets.** Take the tickets that are not `done` in number order: each builds only on the tickets before it. For each, set its **Status** to `doing`, then dispatch one implementer sub-agent, in the foreground, with the paths of the ticket, `milestone.md` and `spec.md`, and this brief:

   > Build this ticket. First read the earlier tickets in its milestone folder and the tickets its **Depends on** names, their Notes, and the commits their **Commit** lines name. Invoke the `tdd` skill, testing at the seams the ticket's acceptance criteria name. Build its Design patterns rows at their Where. The milestone check runs the full suite after the review's fixes, so finish on the unit suite and your e2e tests. Then fill the ticket's Notes with what its diff can't show the next ticket (decisions the ticket left open, departures from it, gotchas you hit) in a few bullets, or "None". Commit everything by invoking the `commit` skill with the spec folder. Then write the commit's SHA into the ticket's **Commit** line and set its **Status** to `done`. Report the commit, and anything you could not build.

   When the ticket was already `doing` or `blocked`, an earlier build stopped on it: add to the brief that part of its work may already be committed on this branch, and that its Notes say what blocked it. When a sub-agent could not build its ticket, set its **Status** to `blocked`, write what stopped it into its Notes, report that to the user and stop: the tickets after it may depend on it. Done when every ticket in the milestone is `done` and committed.

5. **Review.** When the milestone folder holds a `review.md` whose **Head** is the branch's `HEAD`, the milestone as it stands is already reviewed: take that report. Otherwise invoke the `review-changes` skill with the fixed point and the milestone folder; it saves its report as the milestone's `review.md`. Show the user the report. When it flags findings **escalate** that have no **Answer** line, write `ask` into the **Phase** line, then ask the user in one round, in `brainstorm`'s [round format](../brainstorm/SKILL.md#round-format), headed `Escalations`: each finding is a question whose options are fix it as proposed, fix it another way, or skip it, with your recommendation marked on each. Write each answer under its finding as `**Answer:** <answer>`. Done when every escalated finding has its Answer, or none was flagged.

6. **Fix.** When `review.md` holds nothing to fix, skip this. Otherwise dispatch one implementer sub-agent with the path of `review.md` and this brief:

   > Fix the findings in this review: every one not flagged escalate, and each escalated one as its answer says, skipping any the branch already fixes. Verify only the findings you fix. The milestone check runs the full suite next, so finish on the unit suite and your e2e tests. Commit by invoking the `commit` skill with the spec folder.

   The review runs once per milestone, so fixing never turns into a review loop. Done when the fixes are committed, or there was nothing to fix.

7. **Check the increment.** Run the full suite and the check in `milestone.md`'s **Done when**: this is the milestone's one full run. When either fails, dispatch an implementer sub-agent with the failure, committing by the `commit` skill with the spec folder, then run both again. After two failed fixes, write `failed` into the **Phase** line, report the failure to the user and stop. Done when both pass.

8. **Squash into the spec.** Check out the spec's branch. When `git diff sdd/<spec-folder>/spec sdd/<spec-folder>/M<NNN>` is empty, an earlier build already committed the squash: take `HEAD` as its commit. Otherwise run `git merge --squash sdd/<spec-folder>/M<NNN>` and commit it by invoking the `commit` skill with the spec folder: the subject is the milestone's increment in the present tense, the body one to three bullets. Write the commit's SHA into `milestone.md`'s **Squash** line. Keep the milestone's branch: its ticket commits stay readable there until the spec lands. Done when the squash is committed and recorded.

9. **Hand over.** Report the tickets built with their commits, what the review fixed, the squash commit, and the next step: run `/build <spec-folder>` again, for the next milestone or, after the last, to land the spec. Done when the user has the report.

## Land the spec

When no milestone is open, the spec's branch lands on the base as one commit per milestone, with no merge commit.

1. **Check it has not landed.** Find the base branch by **Branches**. When `git merge-base --is-ancestor sdd/<spec-folder>/spec <base>` succeeds, the spec has landed: skip to step 3. Done when you know the spec has not landed.

2. **Land it.** Ask the user before changing the base. When `git status --porcelain` is not empty, report it and stop. Run `git rebase <base> sdd/<spec-folder>/spec`; on a conflict, run `git rebase --abort`, report the conflicting files and stop. Run the full suite on the rebased branch. When it fails, the base's new commits broke the spec: dispatch an implementer sub-agent with the failure, committing by the `commit` skill with the spec folder, then run the suite again; after two failed fixes, report the failure and stop, leaving the base untouched. Then check out the base and run `git merge --ff-only sdd/<spec-folder>/spec`. Done when the base holds the spec's commits, or the user declined.

3. **Record the landing.** Write `**Landed:** <SHA>` into `spec.md`, under its **Base** line, with the SHA of `git rev-parse sdd/<spec-folder>/spec`: specs that depend on this one wait for that line. Skip this when the user declined in step 2, or the line is already there. Done when `spec.md` holds the **Landed** line, or the user declined.

4. **Clean up and hand over.** Offer to delete the spec's branches (`git branch -D` on each of `git branch --list 'sdd/<spec-folder>/*'`); delete them when the user agrees. Report the commits now on the base, and that the spec is built. Done when the user has the report.
