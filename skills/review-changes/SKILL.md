---
name: review-changes
description: Review changes against this repo's coding standards and their spec. Use when the user wants a branch, a PR, or work in progress reviewed.
argument-hint: "[fixed-point] [spec-path]"
metadata:
  type: process
---

Review the diff between `HEAD` and a fixed point on two axes:

- **Standards**: does the code follow this repo's documented coding standards?
- **Spec**: does the code faithfully build what its spec or ticket asked for?

Each axis runs in its own sub-agent and keeps its own heading in the report, so a clean pass on one never masks findings on the other.

## Process

1. **Pin the fixed point.** Take the one the user gave (a commit SHA, branch, tag, `main`, `HEAD~5`). Given a PR, check that the working tree is clean, check out its branch (`gh pr checkout <number>`), and take its base branch as the fixed point. Without either, ask. Check that it resolves (`git rev-parse <fixed-point>`) and that `git diff <fixed-point>...HEAD` (three dots, against the merge-base) is non-empty, so a bad ref fails here rather than inside two sub-agents. Done when you hold the diff command and the commit list from `git log <fixed-point>..HEAD --oneline`.

2. **Find the spec.** Take the path the user gave: a spec folder, a milestone folder, or a ticket file. Otherwise, look under `.sdd/specs/` for the spec folder whose `spec.md` **Key** matches the commit messages' key, or whose name follows `sdd/` in the branch name (an `sdd/<spec-folder>/M<NNN>` branch builds that spec's milestone numbered `NNN`), and the tickets whose **Commit** is in the commit list. Without either, ask the user; when there is none, the Spec axis is skipped. Done when you know what the diff is reviewed against:

   - **Tickets**, when the diff builds one ticket or milestone: their acceptance criteria and Design patterns rows, with `spec.md` read for context only.
   - **The spec**, when the diff builds all of it: every user story, plus its Testing Decisions seams and Design Patterns rows.

3. **Find the standards.** List every file in the repo that documents how code should be written; `docs/STRUCTURE.md`, `CLAUDE.md` and `AGENTS.md` are on it when they exist, and so are the [domain docs](../domain-modeling/DOMAIN-DOCS.md) of the contexts the diff touches. Add the **smell baseline** in [SMELLS.md](SMELLS.md), which applies even when the repo documents nothing. Done when the list is complete.

4. **Run both sub-agents.** Issue both calls together, in the foreground.

   The **Standards** prompt goes to the `alper-skills:reviewer` agent and carries the diff command, the commit list, the standards files, and the full text of `SMELLS.md` (the sub-agent has no other access to it), with this brief: "Report, per file and hunk, (a) every place the diff breaks a documented standard, citing the file and the rule, and (b) every baseline smell you spot, named, with the hunk quoted. Mark each finding as a hard violation or a judgement call, and flag a judgement call **escalate** when its fix would contradict an ADR, change an interface that exists at the fixed point and that code outside the diff calls, or need the spec changed, a seam the spec names included. Skip anything tooling enforces. One or two lines per finding, and report every finding."

   The **Spec** prompt goes to the `alper-skills:spec-reviewer` agent and carries the diff command, the commit list, and the paths from step 2, with this brief: "Report (a) requirements that are missing or partial; (b) behaviour in the diff nobody asked for (scope creep); (c) requirements that look built but built wrong. Treat each Design Patterns row as a requirement: its Pattern, built at its Where. Quote the requirement for each finding. Read an ambiguous requirement as the user stories best fit, and state that reading in its finding. Flag a finding **escalate** when resolving it needs the spec or a ticket changed, not only the code: the requirement contradicts another, or cannot be built as written. One or two lines per finding, and report every finding."

   When step 2 found no spec, run only the Standards sub-agent. Done when every sub-agent you ran has reported.

5. **Report.** Present the reports under `## Standards` and `## Spec`, verbatim or lightly cleaned, each axis's findings in its own sub-agent's order. When the Spec axis was skipped, say so under its heading. End with one line: the number of findings per axis, how many are flagged escalate, and the worst issue within each axis. Done when the user has the report.

6. **Save it.** When step 2 found a spec, save the report as `review.md`, replacing any already there: in the milestone folder when the diff is reviewed against tickets of one milestone, otherwise in the spec folder. Head it `# Review: <milestone or spec title>`, then a `**Fixed point:**` and a `**Head:**` line holding the SHAs `git rev-parse` gives for each. Without a spec, the report stays in chat. Done when `review.md` holds the report, or there is no spec.
