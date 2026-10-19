---
name: to-spec
description: "Turn the current conversation into a spec saved under .scratch/specs/: no interview, just synthesis of what you've already discussed."
disable-model-invocation: true
---

Turn the current conversation and your understanding of the codebase into a spec. This is **synthesis**: every decision in the spec comes from what the user has already said, apart from the seams, and the patterns when the conversation never weighed them, which you propose and the user confirms. When the conversation left a decision open, record it as an open question under Further Notes; the spec states only what was settled.

## Process

1. **Explore.** Read the code the feature touches, unless you already have, plus `GLOSSARY.md` and the ADRs under `docs/adr/` when present. Write the spec in the glossary's terms and within the ADRs' decisions; without them, use the terms the conversation settled. Done when you can name every module the feature builds or changes.

2. **Sketch the seams.** Pick the seams where tests will prove the feature works. Prefer existing seams to new ones, and the highest seam that still reaches the behaviour; propose a new seam at the highest point you can. Fewer is better; one is ideal. Done when every behaviour discussed is reachable from a listed seam.

3. **Record patterns.** Take the pattern choices the conversation settled, including a decision that none is needed. When the conversation never weighed patterns, invoke the `design-patterns` skill and choose them by its rules. Done when each choice has its Pattern, Where and Why, or the spec states that none is needed.

4. **Confirm.** Show the user the seams, plus any pattern choices you made in step 3, and wait. Done when the user confirms them.

5. **Write the spec.** Save it to `.scratch/specs/NNN-<slug>.md`: `NNN` is one past the highest number already there (starting at `001`), and `<slug>` is the spec title in kebab-case. Fill [SPEC-TEMPLATE.md](SPEC-TEMPLATE.md). Done when every section is filled from the conversation and the user has the spec's path.
