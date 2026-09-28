---
name: brainstorm
description: Thinking partner for any topic — interviews you with a frontier-tree loop, dispatching subagents to fact-find when needed. If the topic converges into a feature-shaped scope, continues into the spec-driven path (feature slug + glossary/ADR entries) as phase 1 of `plan`/`implement`; otherwise ends purely conversational, nothing written to disk. Only invoke when the user explicitly runs /brainstorm.
disable-model-invocation: true
---

# Brainstorm

A general-purpose interview loop for thinking through anything — an idea, a decision, an architecture question, a feature to build. Doubles as phase 1 of the spec-driven workflow (`brainstorm` → `plan` → `implement`) exactly when the topic turns out to be a feature; stays purely conversational otherwise.

## The loop

Model the conversation as a **design tree**: every open question branches into the questions that hang off its answer. Work it in **rounds**.

1. **Compute the frontier** — every question whose prerequisites are already answered (by the user, or by a fact you can look up yourself). Don't ask about anything downstream of a still-open question; that belongs to a later round.
2. **Find facts yourself.** If a question needs something derivable from the codebase, existing docs, the web, or a quick lookup, go find it — dispatch a subagent if it's non-trivial. Never ask the user something you could have checked.
3. **Ask the whole frontier in one round.** Number each question, state it plainly, and give your own recommended answer — don't leave it open-ended. Format:

   ```
   ❓ **Q1** - **<title>**: <question>

   ➡️ <your recommended answer>
   ```

4. **Wait for the user's answers.** Their answers reshape the tree — settled branches unblock new questions. Recompute the frontier and ask the next round.
5. **Stop when the frontier is empty** — every branch visited, nothing silently assumed.

Keep rounds proportional to the topic. A small topic might converge in one round; don't manufacture questions to fill a ceremony.

## Ending the session

Once the frontier is empty, ask the user directly which path this was: **a feature to build**, or **purely exploratory**. This is the user's call, not a judgment call to infer — the two paths commit to different amounts of disk state, and guessing wrong either writes unwanted docs or silently drops a feature scope the user meant to capture. Default your own recommendation to exploratory when the topic's shape is genuinely ambiguous — the lower-commitment path is the safer wrong guess.

**Feature to build:**

1. **Propose a feature slug** (kebab-case, short) for the scope just discussed. Confirm it with the user — this slug is what `plan` and `implement` will look for.
2. **Propose glossary/decision entries**, if any surfaced during the interview:
   - A **term** worth recording: a name that's ambiguous, contested, or load-bearing enough that the codebase should agree on one meaning.
   - A **decision** worth recording: a hard-to-reverse or surprising choice with a real trade-off — not every choice, just ones someone will later ask "why did we do it this way?" about.
   - Present proposed entries to the user; only write them once confirmed.
3. **Write confirmed entries**:
   - `docs/GLOSSARY.md` — one growing, append-only file at the project root. Each entry is a `### <term>` section with a short definition. Create the file (with a one-line header) only on first real entry.
   - `docs/ADRS.md` — one growing, append-only file at the project root. Each entry is a `### <date> — <title>` section: context, decision, consequences. Create the file only on first real entry.
   - Append new entries; never rewrite or reorder existing ones.

Do not proceed to writing `spec.md`/`design.md` — that's `plan`'s job. This skill's output is the confirmed slug plus whatever docs entries were written.

**Purely exploratory:** summarize the shared understanding reached. No slug, no docs, no artifact. Nothing gets written to disk.
