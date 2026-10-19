---
name: code-review
description: Review the changes since a fixed point (commit, branch, tag, or merge-base) on two axes, Standards and Spec, in parallel sub-agents. Use when the user wants to review a branch, a PR, or work-in-progress changes.
argument-hint: "[fixed-point] [spec-path]"
metadata:
  type: process
---

Review the diff between `HEAD` and a fixed point on two axes:

- **Standards**: does the code follow this repo's documented coding standards?
- **Spec**: does the code faithfully build what its spec or ticket asked for?

Each axis runs in its own sub-agent, so neither pollutes the other's context, and the report keeps them apart: code can follow every standard and build the wrong thing, or build the right thing against every convention, and keeping the axes apart stops one from masking the other.

## Process

1. **Pin the fixed point.** Take the one the user gave (a commit SHA, branch, tag, `main`, `HEAD~5`); without one, ask. Check that it resolves (`git rev-parse <fixed-point>`) and that `git diff <fixed-point>...HEAD` (three dots, against the merge-base) is non-empty, so a bad ref fails here rather than inside two sub-agents. Done when you hold the diff command and the commit list from `git log <fixed-point>..HEAD --oneline`.

2. **Find the spec.** Take the path the user gave: a spec folder, a milestone folder, or a ticket file. Otherwise, look under `.scratch/specs/` for the spec folder whose slug matches the branch name or the commit messages, and the tickets the commits name. Without either, ask the user; when there is none, the Spec axis is skipped. Done when you know what the diff is reviewed against:

   - **Tickets**, when the diff builds one ticket or milestone: their acceptance criteria and Design patterns rows, with `spec.md` read for context only.
   - **The spec**, when the diff builds all of it: every user story, plus its Testing Decisions seams and Design Patterns rows.

3. **Find the standards.** List every file in the repo that documents how code should be written; `CODING_STANDARDS.md`, `CONTRIBUTING.md`, `CLAUDE.md` and `AGENTS.md` are on it when they exist, and so are `GLOSSARY.md` (names in the diff use its terms) and the ADRs under `docs/adr/` when present. Add the **smell baseline** in [SMELLS.md](SMELLS.md), which applies even when the repo documents nothing. Done when the list is complete.

4. **Run both sub-agents.** Issue both calls together, in the foreground.

   The **Standards** prompt carries the diff command, the commit list, the standards files, and the full text of `SMELLS.md` (the sub-agent has no other access to it), with this brief: "Report, per file and hunk, (a) every place the diff breaks a documented standard, citing the file and the rule, and (b) every baseline smell you spot, named, with the hunk quoted. Mark each finding as a hard violation or a judgement call. Skip anything tooling enforces. Under 400 words."

   The **Spec** prompt carries the diff command, the commit list, and the paths from step 2, with this brief: "Report (a) requirements that are missing or partial; (b) behaviour in the diff nobody asked for (scope creep); (c) requirements that look built but built wrong. Treat each Design Patterns row as a requirement: its Pattern, built at its Where. Quote the requirement for each finding. Under 400 words."

   When step 2 found no spec, run only the Standards sub-agent. Done when every sub-agent you ran has reported.

5. **Report.** Present the reports under `## Standards` and `## Spec`, verbatim or lightly cleaned, each axis's findings in its own sub-agent's order. When the Spec axis was skipped, say so under its heading. End with one line: the number of findings per axis, and the worst issue within each axis. Done when the user has the report.
