---
name: domain-modeling
description: Shared logic for sharpening terminology and recording architectural decisions in the spec-driven framework's docs. Invoked explicitly by `brainstorm` and `spec` the moment a term or a hard-to-reverse decision resolves — never on its own.
---

# Domain Modeling

Invoked by `brainstorm` (during the interview loop) and `spec` (while writing `design.md`'s Approach) at the moment something crystallizes — not batched, not at session end. This skill never decides *when* to fire; the caller decides that and hands over what just resolved. This skill's job is: challenge it, confirm it, write it, immediately.

## Two kinds of entry

- A **term** worth recording: a name that's ambiguous, contested, or load-bearing enough that the codebase should agree on one meaning.
- A **decision** worth recording (an ADR): only when all three hold —
  1. **Hard to reverse** — the cost of changing your mind later is meaningful.
  2. **Surprising without context** — a future reader will wonder "why did they do it this way?"
  3. **The result of a real trade-off** — there were genuine alternatives and one was picked for specific reasons.

  If any of the three is missing, it's not an ADR. Most decisions made during a session don't qualify — skip them silently, don't ask the user to confirm a non-decision.

## Before writing

- **Challenge against the existing glossary**: if the term conflicts with `docs/GLOSSARY.md`, surface the conflict before recording anything new — "the glossary defines X as A, but this sounds like B. Which is it?"
- **Sharpen fuzzy language**: if the term as stated is vague or overloaded, propose a precise canonical name rather than recording the vague one.
- Present the proposed entry (term or ADR) to the user and get confirmation before writing — this happens right there in the conversation, not deferred.

## Writing

- `docs/GLOSSARY.md` — one growing, append-only file at the project root. Format in [references/GLOSSARY-FORMAT.md](./references/GLOSSARY-FORMAT.md). Create the file (one-line header) only on first real entry. Append; never rewrite or reorder existing entries.
- `docs/adr/NNNN-<slug>.md` — one file per decision. Format, numbering, and directory-creation rules in [references/ADR-FORMAT.md](./references/ADR-FORMAT.md). Never edit a past ADR's decision text — a change of mind gets a new ADR that marks the old one superseded.

## Returning control

Once written (or once determined nothing qualifies), hand control straight back to the caller's own flow — this skill doesn't manage the interview loop or the design.md structure, only the docs write.
