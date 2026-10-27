---
name: to-spec
description: Turn the current conversation into a spec saved under .sdd/specs/.
disable-model-invocation: true
metadata:
  type: process
---

Turn the current conversation and your understanding of the codebase into a spec. This is **synthesis**: the spec states only what the conversation settled. You propose two things for the user to confirm: the seams, and the pattern choices when the conversation never weighed patterns. A decision the conversation left open goes under Further Notes as an open question.

This needs `.sdd/settings.json`; when it is missing, tell the user to run `/setup-repository` and stop.

## Process

1. **Explore.** Read the code the feature touches, unless you already have, plus the [domain docs](../domain-modeling/DOMAIN-DOCS.md); without a glossary, use the terms the conversation settled. Done when you can name every module the feature builds or changes.

2. **Settle the dependencies.** Follow [IN-FLIGHT.md](IN-FLIGHT.md) for the modules from step 1; when the conversation already ran it, take its findings and check only the modules it did not cover. Each overlapping spec is a **dependency**: this spec is built after it lands. For each dependency, list the **assumptions** this spec makes about its planned state: an interface, a seam, a behaviour, a term. Follow the **Depends on** lines of each dependency's `spec.md`, and theirs in turn; when one leads back to this spec, report the cycle and drop that dependency. Done when every dependency has its assumptions, and no dependency leads back to this spec.

3. **Sketch the seams.** Pick the seams where tests will prove the feature works. Prefer existing seams to new ones, and the highest seam that still reaches the behaviour; propose a new seam at the highest point you can. Fewer is better; one is ideal. Done when every behaviour discussed is reachable from a listed seam.

4. **Record patterns.** Take the pattern choices the conversation settled, including a decision that none is needed. When the conversation never weighed patterns, invoke the `design-patterns` skill and choose them by its rules. Done when each choice has its Pattern, Where and Why, or the spec states that none is needed.

5. **Confirm.** Show the user the seams, plus any pattern choices you made in step 4, the dependencies from step 2 with their assumptions, and the open questions, each with your recommended answer for the user to settle now or leave open. When `.sdd/settings.json` sets a `commitKeyPattern`, ask for the spec's **key**, the optional prefix every commit for this spec carries (such as `FUNC-1234`). Done when the user confirms them, has settled or left open each open question, and gives a key that fully matches the pattern, or none.

6. **Write the spec.** Save it as `spec.md` in a new spec folder, `.sdd/specs/NNN-<slug>/`: `NNN` is one past the highest folder number already there (starting at `001`), and `<slug>` is the spec title in kebab-case. Fill [SPEC-TEMPLATE.md](SPEC-TEMPLATE.md). Done when every section is filled from the conversation, and the user has the spec's path and the next step, `/to-tickets <spec-folder>`.
