---
name: brainstorm
description: Thinking partner for any topic — interviews you with a frontier-tree loop, dispatching subagents to fact-find when needed. If the topic converges into a feature-shaped scope, continues into the spec-driven path (feature slug + glossary/ADR entries captured inline via `domain-modeling`) as phase 1 of `spec`/`implement`; otherwise ends purely conversational, nothing written to disk. Only invoke when the user explicitly runs /brainstorm.
disable-model-invocation: true
---

# Brainstorm

A general-purpose interview loop for thinking through anything — an idea, a decision, an architecture question, a feature to build. Doubles as phase 1 of the spec-driven workflow (`brainstorm` → `spec` → `implement`) exactly when the topic turns out to be a feature; stays purely conversational otherwise.

## The loop

Model the conversation as a **design tree**: every open question branches into the questions that hang off its answer. Work it in **rounds**.

1. **Compute the frontier** — every question whose prerequisites are already answered (by the user, or by a fact you can look up yourself). Don't ask about anything downstream of a still-open question; that belongs to a later round.
2. **Find facts yourself.** If a question needs something derivable from the codebase, existing docs, the web, or a quick lookup, go find it — dispatch a subagent if it's non-trivial. Never ask the user something you could have checked.
3. **Ask the whole frontier in one round.** Number each question, state it plainly, and give your own recommended answer — don't leave it open-ended. Format:

   ```
   ❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

   ➡️ <your recommended answer>

   ---

   ❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

   ➡️ <your recommended answer>
   ```

4. **Wait for the user's answers.** Their answers reshape the tree — settled branches unblock new questions. Recompute the frontier and ask the next round.
5. **Stop when the frontier is empty** — every branch visited, nothing silently assumed.

Keep rounds proportional to the topic. A small topic might converge in one round; don't manufacture questions to fill a ceremony.

## Capturing terms and decisions as they happen

Don't wait for the session to end. The moment a round settles a term that's ambiguous/contested/load-bearing, or lands on a choice that's hard to reverse, surprising, and the result of a real trade-off, invoke the `domain-modeling` skill right there, mid-round, handing it what just resolved. It challenges/sharpens, confirms with the user, and writes `docs/GLOSSARY.md` / `docs/adr/NNNN-*.md` immediately — don't collect candidates to propose in a batch later. Most rounds won't produce anything that qualifies; that's fine, `domain-modeling` is the one deciding whether a given decision clears the ADR bar, not this loop.

## Ending the session

Once the frontier is empty, ask the user directly which path this was: **a feature to build**, or **purely exploratory**. This is the user's call, not a judgment call to infer — the two paths commit to different amounts of disk state, and guessing wrong either writes unwanted docs or silently drops a feature scope the user meant to capture. Default your own recommendation to exploratory when the topic's shape is genuinely ambiguous — the lower-commitment path is the safer wrong guess.

**Feature to build:**

1. **Propose a feature slug** (kebab-case, short) for the scope just discussed. Confirm it with the user — this slug is what `spec` and `implement` will look for.
2. Any glossary/ADR entries were already written inline during the loop (see above). If something worth recording only becomes obvious in hindsight while wrapping up, invoke `domain-modeling` once more here — but this is the exception, not the normal path.

Do not proceed to writing `spec.md`/`design.md` — that's `spec`'s job. This skill's output is the confirmed slug plus whatever docs entries `domain-modeling` wrote along the way.

**Purely exploratory:** summarize the shared understanding reached. No slug, no docs, no artifact. Nothing gets written to disk.
