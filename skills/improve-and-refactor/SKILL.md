---
name: improve-and-refactor
description: Scan the codebase for deepening and refactoring opportunities, show them as a visual HTML report, then brainstorm the one you pick into the spec pipeline.
argument-hint: "[module, area or pain point]"
disable-model-invocation: true
metadata:
  type: process
---

Find where the codebase's shape causes **friction**, and propose refactors that remove it. Candidates come through two lenses:

- **Deepening**: shallow modules merged or reshaped into a deep one, judged in the `codebase-design` skill's vocabulary and principles.
- **Smell**: a Fowler smell from the baseline in [SMELLS.md](../review-changes/SMELLS.md), with its named fix.

Every candidate preserves behaviour: a refactor changes the shape, never what the code does. The aim is testability and code an agent can navigate.

This needs `.sdd/settings.json`; when it is missing, tell the user to run `/setup-repository` and stop.

## Process

1. **Scope.** Deepening pays back through future changes, so look where change happens (YAGNI). When the user named a module, area or pain point, take it. Otherwise read a long stretch of `git log --stat` and rank the **hot spots**: the files and areas that keep changing. When changes are scattered with no hot spot, widen to the whole codebase. Done when you can name the areas to scan, each with why.

2. **Load the vocabulary.** Invoke the `codebase-design` skill, and use its terms exactly in every candidate. Read the [domain docs](../domain-modeling/DOMAIN-DOCS.md) of the contexts the areas touch: modules are named in the glossary's terms ("the Order intake module"), and the candidates respect the ADRs. Done when you know the terms and the ADR decisions that bind the areas.

3. **Explore.** Dispatch one `alper-skills:explorer` agent in the foreground with the areas, the `codebase-design` vocabulary and principles, the full text of `SMELLS.md`, the glossary, and the ADR decisions (it has no other access to them), and this brief:

   > Walk these areas and report where you feel friction. Explore by reading, following each concept to where it lives: where understanding one concept means bouncing between many small modules; where an interface is nearly as complex as its implementation; where logic was extracted for testability but the bugs hide in how it is called; where modules leak across their seams; which code is untested, or hard to test through its interface; and every baseline smell. Put each suspected shallow module through the deletion test: does deleting it concentrate complexity, or only move it? Report each candidate with its lens (deepening or smell), files, the problem in one sentence, the refactor in one sentence, the deletion-test result or the smell's name, the dependency categories from `codebase-design`'s DEEPENING.md, how the tests would change, and a strength: **Strong**, **Worth exploring** or **Speculative**. Report every candidate.

   Drop a candidate an ADR forbids, unless its friction is real enough to reopen that ADR; keep that one, flagged with the ADR and why. Done when every candidate has every field.

4. **Report.** Number the candidates `#1` to `#N`, strongest first; the number is the candidate's handle, so the user picks by it and you refer to it by it from here on. Write the candidates as one self-contained HTML file, `.sdd/refactors/<YYYY-MM-DD-HHmm>.html`, kept with the repo's other pipeline state and out of git. Build it by [HTML-REPORT.md](HTML-REPORT.md). Open it (`start`, `open` or `xdg-open`) and give the user its absolute path. Interfaces wait for step 5: the report names problems and refactors only. List each candidate's number and title in your reply, and ask: "Which of these would you like to explore? Pick by number." Done when the user has picked a candidate, or declined them all.

5. **Brainstorm the pick.** Invoke the `brainstorm` and `domain-modeling` skills, and run the rounds on the candidate, with each domain-modeling challenge entering the current round as a frontier question.

   - **What**: the behaviour that must stay unchanged, the tests that pin it today and those it lacks, and the scope of the refactor.
   - **How**: the deepened module's interface, where its seam sits, what goes behind it, and how each dependency is tested by its category in DEEPENING.md. When the interface is contested, run `codebase-design`'s DESIGN-IT-TWICE.md.
   - A new module named after a concept missing from `GLOSSARY.md` adds the term as it settles.
   - When the user rejects the candidate for a reason a future scan would need, offer an ADR: "Want me to record this as an ADR so future scans don't suggest it again?" Skip the offer for passing reasons ("not now").

   Done when the user confirms the How summary, or rejects the candidate; a rejection skips step 6.

6. **Size the refactor.** Check the confirmed How summary against four tests:

   - **One seam**: the existing tests at one confirmed seam pin the behaviour, or one new test there would.
   - **Local blast radius**: only callers inside the refactored area change; none elsewhere has to move in the same change.
   - **No ADR reopened**: the candidate carried no ADR callout.
   - **One fresh context**: a single builder sub-agent could build it, with no earlier ticket to build on.

   When all four hold, the refactor is **contained**; otherwise it is **staged**. Done when you can name the size and the test that decided it.

7. **Hand over.** Give the user the size, the deciding test, and the next step, so they can override it:

   - **Contained**: `/to-tickets`, straight from this conversation with no spec, cutting one milestone of one ticket, then `/build`.
   - **Staged**: `/to-spec`, then `/to-tickets` (a refactor whose blast radius spans the codebase is cut by its WIDE-REFACTOR.md, expand–contract), then `/build`.

   Done when the user has the next step, or the rejection is recorded.
