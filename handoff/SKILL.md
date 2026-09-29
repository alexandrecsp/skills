---
name: handoff
description: Compact the current conversation into a handoff document for another agent to pick up.
argument-hint: "What will the next session be used for?"
disable-model-invocation: true
---

# Handoff

Write a handoff document summarising the current conversation so a fresh agent can continue the work. Save it to the OS temp / scratchpad directory — not the current workspace.

Include a "suggested skills" section in the document, naming which skills (this repo's or `synced/`) the next agent should invoke via the Skill tool, and why.

Do not duplicate content already captured in other artifacts (`spec.md`, `design.md`, `issues/`, `docs/GLOSSARY.md`, `docs/ADRS/`, `docs/research/`, commits, diffs). Reference them by path instead — this document is the connective narrative between those artifacts, not a copy of them.

Redact any sensitive information — API keys, passwords, personally identifiable information — before writing.

If the user passed arguments, treat them as a description of what the next session will focus on and tailor the doc's emphasis accordingly.
