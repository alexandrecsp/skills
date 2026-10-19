---
name: to-spec
description: "Turn the current conversation into a spec saved under .scratch/specs/, synthesised from what you've already discussed."
disable-model-invocation: true
metadata:
  type: process
---

Turn the current conversation and your understanding of the codebase into a spec. This is **synthesis**: the spec states only what the conversation settled. You propose two things for the user to confirm: the seams, and the pattern choices when the conversation never weighed patterns. A decision the conversation left open goes under Further Notes as an open question.

## Process

1. **Explore.** Read the code the feature touches, unless you already have, plus the **domain docs** when present: `GLOSSARY.md` and the ADRs in `docs/adr/`; when a `GLOSSARY-MAP.md` exists, the root ADRs plus the `GLOSSARY.md` and `docs/adr/` of each context the feature touches. Write the spec in the glossary's terms and within the ADRs' decisions; without them, use the terms the conversation settled. Done when you can name every module the feature builds or changes.

2. **Sketch the seams.** Pick the seams where tests will prove the feature works. Prefer existing seams to new ones, and the highest seam that still reaches the behaviour; propose a new seam at the highest point you can. Fewer is better; one is ideal. Done when every behaviour discussed is reachable from a listed seam.

3. **Record patterns.** Take the pattern choices the conversation settled, including a decision that none is needed. When the conversation never weighed patterns, invoke the `design-patterns` skill and choose them by its rules. Done when each choice has its Pattern, Where and Why, or the spec states that none is needed.

4. **Confirm.** Show the user the seams, plus any pattern choices you made in step 3, and wait. Done when the user confirms them.

5. **Write the spec.** Save it as `spec.md` in a new spec folder, `.scratch/specs/NNN-<slug>/`: `NNN` is one past the highest folder number already there (starting at `001`), and `<slug>` is the spec title in kebab-case. Fill [SPEC-TEMPLATE.md](SPEC-TEMPLATE.md). Done when every section is filled from the conversation, and the user has the spec's path and the next step, `/to-tickets <spec-folder>`.
