---
name: implement
description: Implement phase of the spec-driven workflow. Executes a feature's design.md step by step (or in parallel waves via git worktrees), following TDD where a step's behavior warrants it, then runs one holistic review of the whole spec before finishing. Only invoke when the user explicitly runs /implement.
disable-model-invocation: true
---

# Implement

Phase 3 of the spec-driven workflow (`brainstorm` → `spec` → `implement`). Executes a feature's `design.md` checklist, one step at a time or in dependency-ordered parallel waves, then reviews the whole feature once before calling it done.

## The one hard gate

Take the feature slug as an argument, or use the most recently modified `.scratch/` directory if none is given. **If `.scratch/<feature>/design.md` doesn't exist, refuse and point the user at `/spec`.** Every other phase degrades gracefully; this one doesn't — no code without a written plan.

Read `spec.md` and `design.md` in full before doing anything.

## Starting the feature branch

Before executing any step: create branch `feat/<feature-slug>/main` (skip creation if it already exists — a resumed run) and commit `spec.md` and `design.md` on it via the `commit` skill. This is the feature branch every step below commits onto (sequential mode) or branches worktrees from (parallel mode).

Note the `/main` — parallel mode's step branches live at `feat/<feature-slug>/step-<id>`, and git branch refs are hierarchical paths: a branch literally named `feat/<feature-slug>` would block any `feat/<feature-slug>/...` branch from being created (can't have a ref be both a file and a directory). Keeping the main branch under the same `feat/<feature-slug>/` prefix avoids that collision.

## Writing the brief

Also before executing any step, write `.scratch/<feature>/brief.md` once — a condensed digest so every step stops re-reading `spec.md`, `design.md`, `docs/GLOSSARY.md`, `docs/ADRS.md`, and a full `implementation-approaches` file from scratch. Include only what's actually load-bearing for this feature:

- The relevant excerpt(s) from whichever `implementation-approaches` file(s) apply — the conventions this feature must follow, not the whole file.
- Any glossary terms or ADRs from `docs/` that are actually relevant to this feature (not the whole growing file).
- The project's commands: test runner, lint, build, typecheck — so every step and the final review run the same things.

Point every step (and the spec-wide review at the end) at `brief.md` instead of re-embedding this material each time.

## TDD for behavioral steps

Before implementing a step, judge whether its referenced `scenarios` describe genuine observable behavior (business logic, a user-facing outcome, anything a Given/When/Then can meaningfully assert on) or whether the step is config/plumbing/infra with nothing behavioral to assert (wiring a dependency, adding a config key, a pure type/schema change with no logic). When genuinely unsure which it is, default to behavioral — a test that turns out to verify very little costs less than logic that ships with no test at all.

- **Behavioral step → follow red/green**: write the test(s) for the step's scenario(s) first, run them and confirm they fail for the right reason (not a typo or missing import), then implement the minimum needed to make them pass, then run the full suite again to confirm green. Refactor only once green, keeping the suite green throughout.
- **Non-behavioral step → skip TDD**: implement directly, then validate with the project's build/lint/typecheck from `brief.md`. Forcing a test-first cycle onto a step with no real behavior to assert produces a test that verifies nothing.

If a step you judged non-behavioral turns out to have real logic once you're in it, switch to red/green rather than finishing it untested — the judgment call is a starting guess, not a commitment.

## Sequential mode (default)

For each unchecked step, in dependency order:

1. Implement the step per "TDD for behavioral steps" above, using `brief.md` instead of re-reading the full source docs. While writing code, let the relevant approach skill (`implementation-approaches`) trigger naturally — it auto-invokes based on what you're building.
2. Once the step's own validation passes (green suite, or build/lint/typecheck for a non-behavioral step): commit by invoking the `commit` skill (automated-run exception applies — see that skill). Then flip that step's `- [ ]` to `- [x]` in `design.md`.

No per-step review gate — see "Spec-wide review" below, which runs once after every step is done.

## Parallel mode (`--parallel`)

Same per-step procedure as Sequential mode above (implement per TDD judgment → validate → commit), plus the machinery to run independent steps concurrently:

1. Parse every step's `depends: [ids]` and `files: [paths]` out of `design.md` and topologically sort into waves — a wave is every step whose dependencies are already merged. The `depends` DAG only models intentional ordering, not accidental overlap: **before scheduling a wave, also check `files` for overlap across the steps you're about to run concurrently.** Two steps in the same candidate wave that touch the same file are a real conflict the DAG doesn't know about — serialize those two (run one, merge it, then the other) even though nothing in `depends` requires it. This is why `design.md` requires a `files` list per step, not just `depends`.
2. Before spawning any worktree, make sure it's actually runnable: worktrees don't get `node_modules` (or your stack's equivalent) for free — link/copy what the project needs into each one before the step subagent starts. Also confirm the project's test runner and `.gitignore` exclude `.scratch/**` — otherwise a full test run picks up every worktree's copy of the tree and multiplies your test count.
3. For each step in the current wave, spawn a subagent in its own git worktree: checkout path `.scratch/<feature>/worktrees/step-<id>/`, on branch `feat/<feature-slug>/step-<id>`, branched from the feature branch. It runs the Sequential mode per-step procedure entirely inside that worktree. **The worker never edits `design.md`** — checkbox mutation happens only in the orchestrator, at merge time (next point), to avoid concurrent writes to the same file from parallel workers.
4. **On a step's success:** back in the orchestrator (not the step's own worker — it can't remove the worktree it's running inside), **squash merge** `feat/<feature-slug>/step-<id>` into the feature branch immediately (`git merge --squash`, then commit via the `commit` skill, automated-run exception applies) — don't hold it for the rest of the wave. Squashing lands exactly one commit per step on `feat/<feature-slug>/main` regardless of how many commits piled up on the step branch (a red/green cycle, a fix, a re-review fix), so the feature branch's history reads one clean commit per step instead of every intermediate one. Then flip that step's `- [ ]` to `- [x]` in `design.md`, then run `git worktree remove .scratch/<feature>/worktrees/step-<id>/` right away, before moving on. Don't batch cleanup for later or leave it for end-of-run; a step isn't done until its worktree is gone. The next wave starts as soon as its actual dependencies are merged, not when the whole current wave finishes.
5. **On a step's failure** (can't reach green after reasonable effort, or an unrecoverable implementation error): quarantine it — leave its worktree and branch in place, unmerged, and record it as failed. Do not block other steps in flight or later waves that don't depend on it.
6. At the end of the run, report: which steps merged, which are quarantined (with their worktree path, branch, and what went wrong) for the user to resolve manually. If anything is quarantined, the spec-wide review below still runs against whatever did merge — say so in the report.

## Spec-wide review

Once every step in `design.md` is checked (or the run ends with some quarantined — see above), review the feature once, holistically, instead of per step:

1. **Dispatch two review subagents in parallel, via the Agent tool** — actually spawn them, waiting synchronously for both to finish (not `run_in_background`) before proceeding. Give each `brief.md`, the full diff of `feat/<feature-slug>/main` against the branch it was created from, and all of `spec.md`:
   - **Spec axis** — does the merged diff, taken as a whole, actually satisfy every scenario in `spec.md` — not just the scenario each step claimed, but the feature end to end.
   - **Quality axis** — is the diff clean across the whole feature (bugs, obvious simplification/reuse misses, inconsistencies introduced by different steps landing independently).
2. On any findings: apply one auto-fix, then one re-review (both axes again). Anything still unresolved after that gets surfaced to the user — don't loop further.
3. This is the only review gate in this skill — there's no per-step equivalent. It trades early, cheap detection (catching a step-4-sized problem before ten more steps build on it) for running the review exactly once per feature instead of once per step. If that trade-off stops paying off on a given feature (a step's `design.md` entry defines a shared contract you're not confident about), that's a signal to strengthen the contract in `spec`'s `## Contracts` section next time, not to silently add a review back in here.

## Progress and resumability

`design.md`'s checklist is the only progress record — no separate progress file. A `/implement` run picks up wherever the checkboxes and any quarantined worktrees left off; work survives a context reset because it's on disk, not in conversation state. The spec-wide review hasn't run until it's been reported to the user — a resumed run that finds all steps already checked still needs to run it if that report never happened.

At the start of a resumed `--parallel` run, sweep `.scratch/<feature>/worktrees/` first: any worktree whose step is already checked `[x]` in `design.md` is a leftover from an interrupted run (its cleanup in point 4 above never ran) — remove it with `git worktree remove` before starting new waves. Anything left over for a step still unchecked is genuinely quarantined; leave it for the failure report, not this sweep.
