---
name: brainstorm
description: Brainstorm a plan or design with the user. Use when the user wants to brainstorm, or to stress-test a plan.
argument-hint: "[topic]"
metadata:
  type: process
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

## What, then How

The tree has two aspects:

1. **What** is the outcome: the problem, who it serves, scope, behaviour, constraints, and what success looks like.
2. **How** is the mechanism: approach, architecture, tools, data, sequencing, the choices that deliver the `What`.

To sort a decision: if two different implementations could both satisfy it, it is a `What`; if it chooses between implementations, it is a `How`.

Every `How` decision hangs off `What` decisions, so settle the `What` first. When a `How` question surfaces during the `What`, **park** it for the `How` phase and keep it out of the round. When the `How` exposes a gap in the `What`, reopen that branch: ask it in the next round tagged **(What)**, and the frontier rule holds back the `How` questions that depend on it.

Close each phase, once its frontier is empty, with a **summary** of its settled decisions as a short bulleted list. The `What` summary also lists the parked questions under **Parked for How**; the user's confirmation of it opens the `How`, with the parked questions in its first frontier.

When the `How` opens, invoke the `design-patterns` skill. Each structural problem in the `How` is then a frontier question: recommend a pattern with its Where and Why, or no pattern when the problem doesn't call for one. The `How` summary lists the settled patterns as Pattern, Where and Why.

## Rounds

Work each phase in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet, including answers to other questions in the same round. Ask the whole frontier in one round, then wait for the user's answers.

Leave no assumption unspoken: any choice you would otherwise make silently becomes a frontier question.

Each round of answers reshapes the tree: settled decisions unblock the questions that hung off them, and new answers can sprout new branches. Recompute the frontier and ask the next round.

### Round format

Head each round with its phase and its number within that phase (`What · Round 2`, `How · Round 1`). Number each question, list its options, and mark your recommendation. When a question has no natural options, give your recommended answer without letters. Word each question so "ok" accepts your recommendation.

```
**What · Round 1**

❓ **Q1** - **<question title>**: <the question>

- **A** - <option>
- **B** - <option>

✅ **B** - <your recommendation and the one-line reason>

---

❓ **Q2** - **<question title>**: <the question>

- **A** - <option>
- **B** - <option>
- **C** - <option>

✅ **A** - <your recommendation and the one-line reason>

---

...

Reply with letters (`1B 2A`), `ok` to take every recommendation, or your own answer.
```

### Facts and Decisions

Finding _facts_ is your job; the user only ever answers _decisions_. When a frontier question needs a fact from the environment (codebase, docs, tools), dispatch a sub-agent to find it. Keep the round moving: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report; ask the rest of the frontier now.

## Done

The session is done when the user confirms the `How` summary: every branch of the `What` and the `How` visited, nothing left silently assumed. That confirmation is the shared understanding; act on the plan only after it. `/to-spec` turns it into a spec.
