---
name: tdd
description: Test-driven development, red then green. Use when the user wants work built test-first, or wants integration tests.
metadata:
  type: process
---

TDD is the **red → green** loop. This skill makes that loop produce tests worth keeping.

Read the **domain docs** when present: `GLOSSARY.md` and the ADRs in `docs/adr/`; when a `GLOSSARY-MAP.md` exists, the root ADRs plus the `GLOSSARY.md` and `docs/adr/` of each context the work touches. Test names and interface vocabulary use the glossary's terms, and the code stays within the ADRs' decisions.

## What a good test is

A good test verifies behaviour through a public interface and reads like a specification: "user can checkout with valid cart" names a capability and survives any refactor that keeps it. The implementation can change entirely while the test stays as it is. See [TESTS.md](TESTS.md) for good and bad examples, and [MOCKING.md](MOCKING.md) for when to mock.

## Seams: where tests go

A **seam** is where a module's interface lives, as the `codebase-design` skill defines it: tests cross it and observe behaviour without reaching inside. Tests live at **confirmed seams** only, which puts the testing effort on the critical paths and complex logic rather than on every edge case.

- When a spec or ticket names the seams (its Testing Decisions or acceptance criteria), they are already confirmed: test there.
- Otherwise, before writing any test, propose the seams, each with a one-line note on what it catches and what it misses, and wait for the user to confirm them. As a sub-agent with no user to ask, report the seams you would propose instead of choosing them.

When the shape of the interface is itself in question (how deep the module is, where the seam belongs, what the interface exposes), invoke the `codebase-design` skill for the vocabulary and principles.

## Anti-patterns

- **Implementation-coupled**: mocks internal collaborators, tests private methods, or verifies through a side channel (querying the database instead of using the interface). The tell: the test breaks on a refactor that kept the behaviour.
- **Tautological**: the expected value is recomputed the way the code computes it, so the test passes by construction. Take expected values from an independent source of truth: a known-good literal, a worked example, the spec.
- **Horizontal slicing**: writing all the tests first, then all the implementation. Bulk tests verify imagined behaviour, test the shape of things, and lock in a test structure before the implementation has taught you anything. Work in **vertical slices**: each test a **tracer bullet** that responds to what the last cycle taught you.

## Rules of the loop

- **Red first.** Write one failing test, run it, and watch it go red for the reason you expect. A test that fails for another reason, or passes, proves nothing yet.
- **Then green.** Write only enough code to pass it, then run the test again.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.
- **One logical assertion per test.**
- **Refactor at review.** Refactoring belongs to the `review-changes` skill's findings, outside the red → green cycle. When nothing will review the work afterwards (`build` reviews each milestone), end by offering the user a `review-changes` run.

## Suites

The **slow tests** are the ones that drive a browser, a device or a deployed service: end-to-end tests, mostly. The **fast suite** is everything else: the linter, the typechecker, and the unit and integration tests. The **full suite** is both. The project's scripts tell them apart.

- **Inside the loop**, run the test you are writing, the fast suite, and only the slow test files your slice touches.
- **Every run is foreground and terse**: under a `timeout`, with a one-line-per-failure reporter, stopping at the first failure, so it hands back its result in one turn and a few lines.

The loop is done when every behaviour in scope has a test at a confirmed seam that went red, then green, and the fast suite and the slow test files you touched pass. Then run the full suite once, unless your brief names a later check that runs it.
