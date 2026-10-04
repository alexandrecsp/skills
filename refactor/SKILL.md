---
name: refactor
description: "Refactor code around a stated problem: investigate, brainstorm the shape with the user, then apply patterns."
disable-model-invocation: true
---

Refactor the code around the problem the user stated. The problem is the **focus**: every finding, question and change answers to it. What you notice outside it goes on a **side list** for step 5. If no focus was given, ask for one before anything else.

## 1. Investigate

Dispatch a sub-agent to map the code the focus touches: its modules, their callers, the seams that exist today, the tests that pin its behaviour, and the patterns the area already applies. Done when every file the focus touches is accounted for and each pain point is anchored to a `file:line`.

Use the `codebase-design` vocabulary (deep, shallow, seam) to name what you find.

## 2. Brainstorm

Run /brainstorm with the user over the investigation. The design tree's root is the focus; its branches are the target shape, where the seams go, what stays, and the order of the moves. Bring the facts from step 1, so the user spends their time on decisions.

Put the patterns on the tree too: judge them through the `patterns` skill from what the investigation found, and bring each candidate as a question with your recommendation. A `none` verdict is valid: the shape may need no named pattern.

Done when the user confirms the shared understanding: a target shape with its pattern verdict, and a sequence of **moves**, each small enough to leave the tests green.

## 3. Pin behaviour

Refactoring preserves behaviour, so the tests are the **tripwire**. Find the tests that cover the focus; where a move would cross uncovered behaviour, write a **characterization test** first, using /tdd at the seams agreed in step 2. Done when every move has a test that goes red if it changes behaviour.

## 4. Apply patterns

Follow the `patterns` references the step 2 verdict names, and apply the moves one at a time, in the agreed order. Run the tripwire after each move; a red test reverts that move. Run typechecking regularly and the full test suite once at the end.

## 5. Close

Use /code-review to review the work, then commit to the current branch with /commit. Report the side list to the user.
