---
name: tdd
description: Red-green-refactor discipline for a single behavioral change — writing the failing test first, the minimal code to pass it, then refactoring only once green. Use when building or fixing behavior test-first, or when `implement` calls it for a step's scenarios.
---

# TDD

TDD is the red → green loop. This skill is the reference that makes that loop produce tests worth keeping: what a good test is, where tests go, the anti-patterns, and the rules of the loop. Every section applies on every cycle: consult before and during the loop, not after.

When exploring the codebase, read `docs/GLOSSARY.md` (if it exists) so test names and interface vocabulary match the project's domain language, and check `docs/ADRS/` for decisions in the area you're touching.

## What a good test is

Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't. A good test reads like a specification: "user can checkout with valid cart" tells you exactly what capability exists, and it survives refactors because it doesn't care about internal structure.

See [references/TESTS.md](references/TESTS.md) for worked examples and [references/MOCKING.md](references/MOCKING.md) for what to mock and how to design for it.

## Seams: where tests go

A **seam** is the public boundary you test at: the interface where you observe behavior without reaching inside. Tests live at seams, never against internals.

**Test only at pre-agreed seams.** Before writing any test, name the seam(s) under test — the step's own `scenarios` (from its `issues/<id>-<slug>.md`, when invoked by `implement`) already imply the seam; when they don't, confirm with the user before writing the test. You can't test everything, so agreeing the seam up front is how testing effort lands on the critical paths and complex logic instead of every edge case.

## Anti-patterns

- **Implementation-coupled**: mocks internal collaborators, tests private methods, or verifies through a side channel (querying the database instead of using the interface). The tell: the test breaks when you refactor but behavior hasn't changed.
- **Tautological**: the assertion recomputes the expected value the way the code does, so it passes by construction and can never disagree with the code. Expected values must come from an independent source of truth: a known-good literal, a worked example, the spec's own scenario.
- **Horizontal slicing**: writing all tests first, then all implementation. Bulk tests verify _imagined_ behavior — you test the _shape_ of things rather than user-facing behavior, and you commit to test structure before understanding the implementation. Work in **vertical slices** instead: one test → one implementation → repeat, each test a **tracer bullet** that responds to what the last cycle taught you.

## Rules of the loop

- **Red before green.** Write the failing test first, run it, and confirm it fails for the right reason (not a typo or missing import) — then write only enough code to pass it. Don't anticipate future tests or add speculative features.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle. Run the full suite again to confirm green before moving to the next slice.
- **Refactor only once green**, keeping the suite green throughout — refactoring never happens mid-cycle, only after a slice is green.

## When invoked by `implement`

Drive the loop against the step's own `scenarios` (its Given/When/Then, from `issues/<id>-<slug>.md`) — each scenario is a slice. Once every scenario is green and the full suite passes, hand control back to `implement`'s own per-step procedure (checkbox flip, commit) — this skill doesn't manage the step's completion, only the red-green-refactor cycles inside it.

## Where it fits

Standalone and reach-for-it-anytime: nothing stops you calling `tdd` for any behavior change outside the spec-driven workflow.

Inside the spec-driven workflow, it's `implement`'s default path for any step judged behavioral (see `implement/SKILL.md`'s "TDD for behavioral steps") — the sibling of `prototype` and `diagnosing-bugs`, except those two are exceptions a step falls into, while this is the step's ordinary main line.
