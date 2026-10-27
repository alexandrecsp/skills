---
name: research
description: Research a question against primary sources in a background sub-agent, and save the cited findings as a Markdown file. Use when the user wants a topic researched, or docs and API facts gathered.
argument-hint: "[question]"
metadata:
  type: process
---

Research runs in a **background sub-agent**, so the conversation keeps moving while it reads. Trust only **primary sources**: the official docs, source code, specs and first-party APIs that own a fact, never a secondary write-up of them.

## Process

1. **Frame the question.** Take the user's question, and sharpen it into the facts that would answer it, each one checkable. When the conversation shows what the answer is for (a `brainstorm` question, a ticket, a bug), carry that purpose along: it decides which facts matter. Done when you can list the facts the research must find.

2. **Pick the file.** When `.sdd/` exists, save to `.sdd/research/<slug>.md`, with `<slug>` the question in kebab-case. Otherwise use the folder where the repo already keeps such notes; with none, ask the user. Done when you hold the path.

3. **Dispatch.** Start one `alper-skills:researcher` agent in the background with the question, the facts from step 1, the purpose, the path, and this brief:

   > Answer this question from primary sources: the official docs, source code, specs and first-party APIs that own each fact. Trace every claim back to the source that owns it; when only a secondary source states it, keep looking, and when the owner cannot be found, mark the claim **unverified**. Pin the versions your sources describe. Write the findings to the path as Markdown: the question, a short answer, then one section per fact with its source linked (a URL, or a file path and line), and an **Open questions** section for what the sources left unanswered. Report the path and the short answer.

   Done when the sub-agent is running; go back to the conversation.

4. **Hand over.** When the sub-agent reports, give the user the path and the short answer, and flag every **unverified** claim and open question. Done when the user has them.
