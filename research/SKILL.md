---
name: research
description: Investigates a question against high-trust primary sources and captures the findings as a Markdown file in `docs/research/`. Use when a topic needs researching, docs/API facts need gathering, or reading legwork can be delegated to a background agent instead of blocking the current work.
---

# Research

Spin up a **background agent** to do the research, so the current work keeps moving while it reads.

Its job:

1. Investigate the question against **primary sources** (official docs, source code, specs, first-party APIs), not a secondary write-up of them. Follow every claim back to the source that owns it.
2. Write the findings to `docs/research/<slug>.md` at the project root, citing each claim's source inline. Create `docs/research/` on first use — each research task gets its own file, unlike the append-only `docs/GLOSSARY.md`.
3. Report back the file path and a one-line summary once done; the findings live in the file, not restated in the conversation.
