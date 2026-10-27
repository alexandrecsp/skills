---
name: pr
description: Write a pull request's title and body as a visual summary, before-and-after evidence, and a merge-danger call, and open or update the PR on request.
argument-hint: "[base] [spec-path]"
disable-model-invocation: true
metadata:
  type: process
---

A PR body shows the reviewer the **shape** of the change before they open the diff, the **evidence** that it works, and the **merge danger**. It opens on its first heading, and its prose is the one sentence each visual or claim needs, in the glossary's terms.

The title reads like a commit title: it completes "If applied, this PR…", in the present tense, capitalised, at most 72 characters, prefixed `<key>: ` when the spec has a key.

```markdown
## Summary

<one sentence>

<visual>

## Evidence

- **<behaviour>**
  **Before:** <screenshot, output, or the failing test as pseudocode>
  **After:** <screenshot, or the passing run's output>

**Unverified:** <behaviours with no before and after, or "None">

## Merge Danger

**Door:** <one-way | two-way>: <why, in one line>

**Blast radius:** <each thing that breaks if this is wrong>
```

When the repo has a PR template (`.github/pull_request_template.md`, or a file in `.github/PULL_REQUEST_TEMPLATE/`), fill that template, with these three sections under its description heading.

## Summary

Pick the **smallest view** that makes the change's point clear: usually one, sometimes two. [VISUALS.md](VISUALS.md) shows each:

- **Pseudocode**: logic or an algorithm.
- **Call tree**: runtime control flow.
- **Component tree**: UI structure, with the state and module boundaries that matter.
- **File tree**: file responsibilities, or a broad refactor.
- **Mermaid**: interaction or data flow between components.
- **Shaped diff**: one of the views above with `+` and `-` lines, when the surrounding shape already exists.
- **Whole block**: most of it is new, or the reviewer needs the target shape.

Keep only the calls, files, props, states and boundaries the reviewer needs to read the diff. When the diff spans milestones, each milestone's increment gets its own sentence and visual.

## Evidence

Each item pairs a **before** with an **after** for one behaviour the PR claims. A screenshot is S-tier, when the change is visual and the environment can take one. Execution is A-tier: the exact test, as pseudocode, that fails on the base and passes on the head, or the console output that changed. Every Before and After quotes a run you did.

## Merge Danger

The **door** is two-way when reverting the merge undoes it, one-way when it does not: destructive data changes and schema migrations, a removed or changed interface that others consume, anything that leaves the system (emails sent, webhooks fired, packages published), and any decision an ADR records as hard to reverse. A change behind a flag stays two-way only until the first write in the new shape lands. When the repo's agent docs list changes it treats as one-way, their list wins. The agent that wrote the change is grading it, so a close call is one-way, with the reason stated.

The **blast radius** names each thing that breaks if the change is wrong, by the module or user that feels it: consumers of a changed interface, data already stored, layout shift, mobile, performance.

## Process

1. **Pin the base.** Take the one the user gave. Otherwise, take the base of the branch's open PR (`gh pr view --json baseRefName`), else the **Base** line of the spec's `spec.md`, else the remote's default branch (`git symbolic-ref --short refs/remotes/origin/HEAD`, without `origin/`). Done when `git diff <base>...HEAD` is non-empty and you hold the commit list from `git log <base>..HEAD --oneline`.

2. **List the claims.** Take the spec path the user gave. Otherwise, look under `.sdd/specs/` for the spec folder whose `spec.md` **Key** matches the commit messages' key, or whose name follows `sdd/` in the branch name, and the tickets whose **Commit** is in the commit list. Read the [domain docs](../domain-modeling/DOMAIN-DOCS.md) of the contexts the diff touches. Done when you can list every behaviour the PR claims: the tickets' acceptance criteria, the spec's user stories when the diff builds all of it, or, without a spec, each behaviour the commit messages name.

3. **Gather evidence.** Run the full suite on the head. For each claim, find the test in the diff that proves it; take a screenshot when the change is visual. For the before, run that test against the base in a worktree (`git worktree add <scratch-dir> <base>`, copy the test in, run it, then `git worktree remove`), or capture the base's output there. Done when every claim from step 2 has a before and an after, or sits under **Unverified**.

4. **Call the merge danger** by **Merge Danger**, weighing every hunk of the diff. Done when every hunk is weighed and the call names each one-way change, or states there is none.

5. **Write it.** Fill the title and body, and show them to the user as one raw markdown block. Done when every section is filled, Summary holds at most two visuals per increment, and every Evidence item quotes a run from step 3.

6. **Open or update the PR**, when the user asked for one. When `gh pr view` finds a PR for the branch, update it: `gh pr edit --title "<title>" --body-file -`. Otherwise push the branch (`git push -u origin HEAD`) and open it: `gh pr create --base <base> --title "<title>" --body-file -`. Done when you have reported the PR's URL; without the ask, done at step 5.
