---
name: implement
description: Implement phase of the spec-driven workflow. Executes a feature's issues/<id>.md files step by step (or in parallel waves via git worktrees), following TDD where a step's behavior warrants it, then runs one holistic review of the whole spec before finishing.
disable-model-invocation: true
---

# Implement

Phase 3 of the spec-driven workflow (`brainstorm` → `spec` → `implement`). Executes a feature's steps — one `issues/<id>.md` file per step — one at a time or in dependency-ordered parallel waves, then reviews the whole feature once before calling it done.

## The one hard gate

Take the feature slug as an argument, or use the most recently modified `.scratch/` directory if none is given. **If `.scratch/<feature>/design.md` doesn't exist, or `.scratch/<feature>/issues/` is missing or empty, refuse and point the user at `/spec`.** Every other phase degrades gracefully; this one doesn't — no code without a written plan.

Read `spec.md` and `design.md` in full, and list every `issues/<id>.md`, before doing anything.

## Starting the feature branch

Before executing any step: create branch `feat/<feature-slug>/main` (skip creation if it already exists — a resumed run) and commit `spec.md`, `design.md`, and the full `issues/` directory on it via the `commit` skill. This is the feature branch every step below commits onto (sequential mode) or branches worktrees from (parallel mode).

Note the `/main` — parallel mode's step branches live at `feat/<feature-slug>/step-<id>`, and git branch refs are hierarchical paths: a branch literally named `feat/<feature-slug>` would block any `feat/<feature-slug>/...` branch from being created (can't have a ref be both a file and a directory). Keeping the main branch under the same `feat/<feature-slug>/` prefix avoids that collision.

## Writing the brief

Also before executing any step, write `.scratch/<feature>/brief.md` once — a condensed digest so every step stops re-reading `spec.md`, `design.md`, `docs/GLOSSARY.md`, `docs/adr/`, and a full `architecture-patterns` file from scratch. Include only what's actually load-bearing for this feature:

- The relevant excerpt(s) from whichever `architecture-patterns` file(s) apply — the conventions this feature must follow, not the whole file.
- Any glossary terms or `docs/adr/NNNN-*.md` entries that are actually relevant to this feature (not every ADR in the directory).
- The project's commands: test runner, lint, build, typecheck — so every step and the final review run the same things.

This skill only reads `docs/GLOSSARY.md`/`docs/adr/` — it never writes to them, even when a step's implementation reveals a hard-to-reverse decision. That belongs to `brainstorm`/`spec` (or a manual pass); recording it here would put ceremony in the middle of executing a step that's supposed to just run.

Point every step (and the spec-wide review at the end) at `brief.md` instead of re-embedding this material each time.

## TDD for behavioral steps

Before implementing a step, judge whether its referenced `scenarios` describe genuine observable behavior (business logic, a user-facing outcome, anything a Given/When/Then can meaningfully assert on) or whether the step is config/plumbing/infra with nothing behavioral to assert (wiring a dependency, adding a config key, a pure type/schema change with no logic). When genuinely unsure which it is, default to behavioral — a test that turns out to verify very little costs less than logic that ships with no test at all.

- **Behavioral step → follow red/green**: write the test(s) for the step's scenario(s) first, run them and confirm they fail for the right reason (not a typo or missing import), then implement the minimum needed to make them pass, then run the full suite again to confirm green. Refactor only once green, keeping the suite green throughout.
- **Non-behavioral step → skip TDD**: implement directly, then validate with the project's build/lint/typecheck from `brief.md`. Forcing a test-first cycle onto a step with no real behavior to assert produces a test that verifies nothing.

If a step you judged non-behavioral turns out to have real logic once you're in it, switch to red/green rather than finishing it untested — the judgment call is a starting guess, not a commitment.

## When a step rests on an unsettled design question

Before implementing a step, check whether its behavior actually depends on a state model or UI direction that `spec.md`/`design.md` gesture at but never nail down — an assumption wearing the clothes of a decision. Don't guess, and don't burn the step's own context arguing about it in place; dispatch the `prototype` skill as a subagent (via the Agent tool), handing it the one-sentence question. It works the question on its own branch, in isolation, and returns only the one-line answer and the branch name — none of the exploration that produced it.

On return: write that answer, question, and branch name as a pointer into `design.md`'s Approach or Contracts section (`prototype` itself may also invoke `domain-modeling` for an ADR, if the decision cleared that bar), then resume the step with the answer in hand.

**Sequential mode**: this only pauses the one blocked step; nothing else in this skill is running concurrently to hold up.

**Parallel mode**: treat it like the existing quarantine mechanic (see [references/PARALLEL-MODE.md](./references/PARALLEL-MODE.md)) — that step's worktree waits without merging or being torn down, independent waves that don't depend on it keep moving, and the step resumes its own per-step procedure in the same worktree once `prototype` returns.

## When a step rests on a bug, not a design question

Before implementing a step, also check whether it's blocked not by an undecided design question but by an actual defect — existing behavior that's wrong, standing in the way of the step rather than something nobody decided yet. Dispatch the `diagnosing-bugs` skill as a subagent (via the Agent tool), handing it the observed symptom. It builds its own red-capable feedback loop, diagnoses, fixes, and adds a regression test on its own footing, then returns with the fix in hand and the hypothesis that turned out correct.

On return: resume the step with the fix already in place — the regression test it added counts toward the step's own validation, it isn't a separate thing to redo.

**Sequential mode**: same as above — pauses only the one blocked step.

**Parallel mode**: same quarantine mechanic as `prototype`'s (see [references/PARALLEL-MODE.md](./references/PARALLEL-MODE.md)) — that step's worktree waits without merging or being torn down, other waves keep moving, and the step resumes once `diagnosing-bugs` returns.

## Sequential mode (default)

For each step whose `issues/<id>.md` is still unchecked, in dependency order:

1. Implement the step per "TDD for behavioral steps" and "When a step rests on an unsettled design question" above, using `brief.md` instead of re-reading the full source docs. While writing code, let the relevant approach skill (`architecture-patterns`) trigger naturally — it auto-invokes based on what you're building.
2. Once the step's own validation passes (green suite, or build/lint/typecheck for a non-behavioral step): flip that step's `- [ ] Done` to `- [x] Done` in its own `issues/<id>.md`, then commit by invoking the `commit` skill (automated-run exception applies — see that skill) — the flip goes in the same commit as the step's own work, not a separate one.

No per-step review gate — see "Spec-wide review" below, which runs once after every step is done.

## Parallel mode (`--parallel`)

Runs the same per-step procedure as Sequential mode, plus the machinery to run independent steps concurrently via git worktrees — waves from the `depends`/`files` DAG, squash-merge on success, quarantine on failure. Read [references/PARALLEL-MODE.md](./references/PARALLEL-MODE.md) before running with `--parallel`; sequential runs never need it.

## Spec-wide review

Once every `issues/<id>.md` is checked (or the run ends with some quarantined — see above), review the feature once, holistically, instead of per step:

1. **Dispatch two review subagents in parallel, via the Agent tool** — actually spawn them, waiting synchronously for both to finish (not `run_in_background`) before proceeding. Give each `brief.md`, the full diff of `feat/<feature-slug>/main` against the branch it was created from, and all of `spec.md`:
   - **Spec axis** — does the merged diff, taken as a whole, actually satisfy every scenario in `spec.md` — not just the scenario each step claimed, but the feature end to end.
   - **Quality axis** — is the diff clean across the whole feature (bugs, obvious simplification/reuse misses, inconsistencies introduced by different steps landing independently).
2. On any findings: apply one auto-fix, then one re-review (both axes again). Anything still unresolved after that gets surfaced to the user — don't loop further.
3. This is the only review gate in this skill — there's no per-step equivalent. It trades early, cheap detection (catching a step-4-sized problem before ten more steps build on it) for running the review exactly once per feature instead of once per step. If that trade-off stops paying off on a given feature (a step's `issues/<id>.md` touches a shared contract you're not confident about), that's a signal to strengthen the contract in `spec`'s `## Contracts` section next time, not to silently add a review back in here.

## Progress and resumability

Each step's own `issues/<id>.md` checkbox is the only progress record for that step — no separate progress file. A `/implement` run picks up wherever the `issues/` checkboxes and any quarantined worktrees left off; work survives a context reset because it's on disk, not in conversation state. The spec-wide review hasn't run until it's been reported to the user — a resumed run that finds all steps already checked still needs to run it if that report never happened.

A resumed `--parallel` run has its own worktree sweep — see [references/PARALLEL-MODE.md](./references/PARALLEL-MODE.md).
