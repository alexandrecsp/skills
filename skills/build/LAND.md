# Land the spec

When no milestone is open, the spec's branch lands on the base as one commit per milestone, with no merge commit.

1. **Check it has not landed.** Find the base branch by **Branches**. When `git merge-base --is-ancestor sdd/<spec-folder>/spec <base>` succeeds, the spec has landed: skip to step 3. Done when you know the spec has not landed.

2. **Land it.** Ask the user before changing the base. When `git status --porcelain` is not empty, report it and stop. Run `git rebase <base> sdd/<spec-folder>/spec`; on a conflict, run `git rebase --abort`, report the conflicting files and stop. Run the full suite on the rebased branch. When it fails, the base's new commits broke the spec: dispatch a fixer with the failure, committing by the `commit` skill with the spec folder, then run the suite again; after two failed fixes, report the failure and stop, leaving the base untouched. Then check out the base and run `git merge --ff-only sdd/<spec-folder>/spec`. Done when the base holds the spec's commits, or the user declined.

3. **Record the landing.** Write `**Landed:** <SHA>` into `spec.md`, under its **Base** line, with the SHA of `git rev-parse sdd/<spec-folder>/spec`: specs that depend on this one wait for that line. Skip this when the user declined in step 2, or the line is already there. Done when `spec.md` holds the **Landed** line, or the user declined.

4. **Clean up and hand over.** Offer to delete the spec's branches (`git branch -D` on each of `git branch --list 'sdd/<spec-folder>/*'`); delete them when the user agrees. Report the commits now on the base, and that the spec is built. Done when the user has the report.
