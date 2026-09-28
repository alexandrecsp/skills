---
name: commit
description: Writes a git commit message whose headline states the effect of applying the commit ("If you apply this commit it will...") and a body listing at most 5 of the most significant changes. Use when committing changes or writing a commit message.
---

# Commit

## Overview

A commit message should tell a reviewer what happens *if they apply it* — not narrate what the author did. This skill produces messages in the imperative "if applied, this commit will..." style (the same convention Git itself uses for its own generated messages), plus a short, prioritized list of the most significant changes — never a full diff recap.

## Process

1. Run `git status` and `git diff --staged` (fall back to `git diff` if nothing is staged, and tell the user you're about to stage it) to see the actual changes. Never write a message from assumption or from the user's description alone if the repo is available.
2. Split into commits that make sense: if the changes touch unrelated concerns (e.g. an unrelated bugfix mixed with a refactor, or config changes mixed with feature work), don't lump them into one commit. Group changes by the single effect each commit should have, `git reset` staged-but-mixed changes if needed, and stage/commit each group separately with `git add <specific files>` — never `git add -A` across unrelated groups. Small, genuinely related changes stay in one commit; don't split for its own sake.
3. For each commit (repeat steps 4-6 below per group):
4. Identify the single most significant effect of the change — this becomes the headline.
5. Identify up to 5 of the most significant individual changes, ranked by importance. If there are fewer than 5 meaningfully distinct changes, list fewer — never pad.
6. Write the message per Output Format.
7. Show the message(s) to the user before committing, unless they've already approved committing in this turn — **or** this skill was invoked from an automated workflow with no one there to approve mid-run (e.g. `implement`'s per-step commits). In that case, the invocation to run the workflow at all is the approval; commit directly and include the message verbatim in that step's report so the user still sees it, just after the fact.

## Output Format

**Headline**: one line, imperative mood, completing the sentence "If you apply this commit it will...". Keep it under ~70 characters where possible. Do not literally include the phrase "If you apply this commit" in the headline itself — write the completion only (e.g. `Fix race condition in session cleanup`), matching Git's own convention for generated commit summaries.

**Body**: a blank line, then a bullet list of at most 5 items, most significant first. Each bullet is a concrete, specific change — not a vague category.

```
<Headline: imperative, states the effect>

- <Most significant change>
- <Next most significant change>
- <...>
```

### Example

```
Fix race condition in session cleanup

- Guard session eviction with a lock so concurrent requests can't double-free
- Add regression test reproducing the interleaving that caused the crash
- Log evicted session IDs at debug level for post-incident tracing
```

## Red Flags

- Headline phrased as what the author did ("Fixed bug", "Added tests") instead of what applying the commit does
- Body describes the diff mechanically (file-by-file) rather than the significant changes
- More than 3 bullets, or bullets padded with trivial/cosmetic changes to reach a count
- Message written without reading the actual diff
- Unrelated changes (different files, different concerns) squashed into a single commit for convenience
- Splitting a single coherent change into multiple commits just to pad commit count
