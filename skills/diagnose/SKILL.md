---
name: diagnose
description: Diagnose a hard bug or performance regression through a red feedback loop. Use when the user asks to diagnose or debug something, or a failure, flake or slowdown survives a first look.
argument-hint: "[symptom]"
metadata:
  type: process
  credits:
    - "Matt Pocock's diagnosing-bugs, https://github.com/mattpocock/skills/tree/main/skills/engineering/diagnosing-bugs"
---

Diagnosis is a loop: a **tight** command that goes **red** on this bug and **green** once it is fixed. Every later step (minimising, hypothesising, instrumenting, fixing) consumes that loop, so the loop is the hard part and the rest is mechanical. Each step below is a gate: the next starts only when the current one's **Done when** holds.

Read the **domain docs** when present: `GLOSSARY.md` and the ADRs in `docs/adr/`; when a `GLOSSARY-MAP.md` exists, the root ADRs plus the `GLOSSARY.md` and `docs/adr/` of each context the bug touches.

## Redact

Every command, output and captured artifact you show has its secrets replaced with `<REDACTED>`. Build loops on env vars, so credentials stay in the environment, and quote only the lines of a capture that carry the signal (captures carry auth headers and cookies). When the redacted output is too thin to diagnose from, say so and ask the user.

## Process

### 1. Build a tight loop

Be **relentless** here. Ways to build one, roughly in order of preference:

1. **Failing test** at whatever seam reaches the bug: unit, integration, e2e.
2. **HTTP script** (curl or similar) against a running dev server.
3. **CLI run** on a fixture input, diffing stdout against a known-good snapshot.
4. **Headless browser script** (Playwright, Puppeteer) asserting on DOM, console or network.
5. **Replayed capture**: a real request, payload or event log saved to disk and run through the code path in isolation.
6. **Throwaway harness**: a minimal slice of the system (one service, mocked deps) that hits the bug path in one call.
7. **Property or fuzz loop**, for "sometimes wrong output": run a thousand random inputs and catch the failure mode.
8. **Bisection harness**, when the bug appeared between two known states (commit, dataset, version): automate "set up state X, check" and hand it to `git bisect run`.
9. **Differential loop**: the same input through the old and new version (or two configs), outputs diffed.
10. **Human-in-the-loop script**, last resort, when a human must click: copy [scripts/hitl-loop.template.sh](scripts/hitl-loop.template.sh), edit its steps, and run it; the user follows its prompts and their answers come back to you as `KEY=VALUE` lines.

Then **tighten** it: faster (cache setup, skip unrelated init, narrow the scope), sharper (assert the exact symptom, beyond "didn't crash"), deterministic (pin time, seed RNG, isolate the filesystem, freeze the network). For a non-deterministic bug the target is a **reproduction rate** high enough to debug against: loop the trigger 100×, parallelise, add stress, narrow timing windows, inject sleeps. A 50% flake is debuggable; a 1% flake needs more.

Code reading in this step serves the loop: finding the seam and the trigger. Theories wait for step 3.

When no loop can be built, stop and report what you tried, then ask the user for one of: access to an environment that reproduces it, a redacted capture (HAR file, log dump, core dump, timestamped screen recording), or permission to add temporary production instrumentation.

Done when you can name **one command** you have already run, with its invocation and output shown, that is:

- **Red-capable**: it drives the real bug path and asserts the user's exact symptom, so it goes red on this bug and green once fixed.
- **Deterministic**: the same verdict every run, or for a flake, a pinned and high reproduction rate.
- **Fast**: seconds, not minutes.
- **Agent-runnable**: it runs unattended, or through the human-in-the-loop script.

### 2. Reproduce and minimise

Run the loop and watch it go red. Confirm the red is the user's failure, not a nearby one (a wrong bug gets a wrong fix), that it holds across runs, and capture the exact symptom (error message, wrong output, timing) for step 5 to verify against.

Then shrink the repro: cut inputs, callers, config, data and steps **one at a time**, re-running the loop after each cut. A minimal repro leaves fewer suspects for step 3 and becomes the regression test in step 5.

Done when the loop is red on the user's symptom and every remaining element is **load-bearing**: removing any one turns the loop green.

### 3. Hypothesise

Rank 3–5 hypotheses before testing any; a single hypothesis anchors on the first plausible idea. Each is **falsifiable**, stated as a prediction: "If X is the cause, then changing Y turns the loop green (or changing Z makes it worse)." A hypothesis whose prediction you cannot state gets sharpened or dropped.

Show the ranked list to the user; they often re-rank it at once ("we just deployed a change to #3") or have ruled some out already. Continue with your own ranking when the user is away.

Done when the user has seen 3–5 ranked hypotheses, each with its prediction.

### 4. Instrument

Each probe tests one prediction from step 3, changing **one variable** at a time. Reach first for a debugger or REPL, where the environment has one; otherwise add targeted logs at the boundaries that tell hypotheses apart. **Tag** every debug log with one unique prefix, such as `[DEBUG-a4f2]`, so cleanup is a single grep.

For a performance regression, measure instead of logging: take a baseline (a timing harness, `performance.now()`, a profiler, a query plan), then bisect against it.

Done when one hypothesis has survived its prediction and the others are falsified: that one is the **root cause**.

### 5. Fix with a regression test

The regression test goes at a **correct seam**: one where the test exercises the bug pattern as it really occurs at the call site, with every caller and step of the chain the minimised repro proved load-bearing. A shallower seam (a single-caller test for a multi-caller bug, a unit test that skips the triggering chain) gives false confidence, so a missing correct seam is itself a finding: record it, and skip the test.

With a correct seam, write the test the `tdd` way:

1. Turn the minimised repro into a failing test at that seam, and watch it go red on the captured symptom. When you forced the red by mutating code or a fixture, `diff` against a pristine copy to prove the mutation landed.
2. Apply the fix, and watch the test go green.
3. Re-run the step 1 loop on the original, un-minimised scenario.

Done when the step 1 loop is green, and the regression test is green or the missing seam is recorded.

### 6. Clean up and report

1. Remove every tagged log (`grep` for the prefix finds none), and delete throwaway harnesses and prototypes.
2. Run the full suite.
3. Commit with the `commit` skill, its details naming the hypothesis that proved right, so the next debugger learns from it.
4. Report to the user: the root cause, the fix, the regression test or the missing seam, and the commit SHA. When the seam was missing, offer `/improve-and-refactor` to deepen the module so the bug can be locked down.

Done when the tag grep is empty, the suite passes, the commit exists, and the user has the report.
