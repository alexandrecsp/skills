# Personal Claude Code Skills

A personal skills framework for [Claude Code](https://claude.com/claude-code), centered on a spec-driven workflow for building features plus a few standalone utilities. Lives at `~/.claude/skills` and applies across all projects.

`synced/` is excluded (see `.gitignore`) — it holds Anthropic's built-in skills (docs, pdf, pptx, xlsx, morning, skill-creator, import-memory), which are managed separately and not part of this personal framework.

## The spec-driven workflow

Three phases, invoked explicitly (`/brainstorm`, `/spec`, `/implement` — none auto-trigger):

```
brainstorm  →  spec  →  implement
(explore)      (spec)     (build)
```

### 1. `brainstorm`
A frontier-tree interview loop for thinking through anything — an idea, a decision, an architecture question. Works in rounds: compute every question whose prerequisites are answered, look up what it can itself (dispatching subagents for non-trivial fact-finding), ask the rest of the user in one batch with a recommended answer, repeat until nothing's left open.

If the topic converges into a feature, it proposes a **feature slug**. Along the way, whenever a term or a hard-to-reverse decision crystallizes, it invokes `domain-modeling` right there to capture it — glossary terms appended to `docs/GLOSSARY.md`, decisions written as new numbered files under `docs/ADRS/` (`0001-<slug>.md`, `0002-<slug>.md`, ...) — instead of batching proposals for the end. If it stays exploratory, nothing is written to disk. Doesn't write `spec.md`/`design.md` itself; that's `spec`'s job.

### 2. `spec`
Turns a converged feature scope into these artifacts under `.scratch/<feature>/`:

- **`spec.md`** — the what/why. Problem, scope, and numbered Given/When/Then scenarios. Meant to stay stable once implementation starts.
- **`design.md`** — the how. Approach (informed by `architecture-patterns`, and by `domain-modeling` when settling it surfaces an ADR-worthy decision), a diagram only when a real trigger applies (3+ collaborating files, a process/network boundary, or a contract easier shown than told), explicit `## Contracts` for any shared interface a step introduces. No step list — that's `issues/`, below.
- **`issues/<id>-<slug>.md`** — one file per step (numeric id + kebab-case slug, e.g. `issues/1-create-user-entity.md`), each with its own `depends`/`scenarios`/`files` metadata (forming a DAG across files) and a `- [ ] Done` checkbox `implement` flips as it works.

Can run standalone without a prior `brainstorm` for well-understood features.

### 3. `implement`
Executes the feature's steps — one `issues/<id>-<slug>.md` per step — refusing to run at all if `design.md` or `issues/` doesn't exist yet. Before touching any step: creates branch `feat/<feature-slug>/main`, commits `spec.md`/`design.md`/`issues/`, and writes a condensed `.scratch/<feature>/brief.md` (relevant approach excerpts, glossary terms, relevant `docs/ADRS/` entries, project commands) so steps stop re-reading full source docs. Read-only with respect to `docs/GLOSSARY.md`/`docs/ADRS/` — it never writes new entries, even when a step reveals a hard-to-reverse decision; that stays `brainstorm`/`spec`'s job via `domain-modeling`.

- **TDD when a step is behavioral**: red → green → refactor. Skipped for pure config/plumbing steps.
- **Sequential mode** (default): one step at a time, flip that step's own `issues/<id>-<slug>.md` checkbox, commit via `commit` after each.
- **Parallel mode** (`--parallel`): topologically sorts the `depends` DAG (read from `issues/*.md`) into waves, checks `files` for conflicts the DAG doesn't know about, runs each step in its own git worktree/subagent. Each step owns its own `issues/<id>-<slug>.md` exclusively, so the worker flips its own checkbox as part of its own commit — no shared-file write conflict, no orchestrator-only mutation step (see `docs/ADRS/0001-per-step-issue-files.md`). Squash-merges carry that flip over on success; on failure, the worker notes the reason in its own `issues/<id>-<slug>.md` before the step is quarantined (left unmerged) without blocking unrelated work.
- **Spec-wide review**: once, after all steps are done — two parallel subagents check the Spec axis (does the whole diff satisfy every scenario) and Quality axis (bugs, simplification, cross-step inconsistency), one auto-fix + re-review round, then surface anything left to the user.
- Progress lives entirely in each step's own `issues/<id>-<slug>.md` checkbox — resumable across sessions with no separate state file.

## Standalone utilities

| Skill | Purpose |
|---|---|
| `commit` | Writes commit messages in "if you apply this commit it will..." style with a short, prioritized bullet list — never a diff recap. Splits unrelated changes into separate commits. Used both directly and internally by `implement`. |
| `domain-modeling` | Shared write logic for `docs/GLOSSARY.md` and `docs/ADRS/`. Never auto-triggers or runs standalone — `brainstorm` and `spec` invoke it inline the moment a term or an ADR-worthy decision (hard to reverse, surprising, a real trade-off) resolves. Challenges fuzzy terms, confirms with the user, then writes immediately. `implement` reads these docs but never calls this skill. |
| `to-diagram` | Builds a Mermaid diagram (sequence, C4, flowchart, or roadmap) from real codebase names only — never invented components. Identifies the type, confirms with the user, builds from the matching template (`sequence.md`, `c4.md`, `flowchart.md`, `roadmap.md`). Used internally by `spec` when a design needs a flow diagram. |
| `architecture-patterns` | Router (not a skill invoked on its own) to house conventions for a given kind of work: `frontend-design.md`, `api-design.md`, `hexagonal-pattern.md`, `unity.md`. Consulted by `spec` when writing the Approach section and left to auto-trigger during `implement` while writing code. |
| `prototype` | Settles one unresolved design question (state model or UI direction) with throwaway code on a never-merged branch. Standalone (`/prototype`) or invoked by `implement` when a step depends on something `spec` never actually settled. |
| `diagnosing-bugs` | Six-phase discipline for hard bugs and perf regressions: build a tight red-capable feedback loop first, then reproduce/minimise, hypothesise, instrument, fix with a regression test, clean up. Standalone (auto-triggers on "debug"/"broken"/"slow") or invoked by `implement` when a step is blocked by an actual defect rather than an unsettled design question. |
| `research` | Delegates reading legwork to a background agent: investigates a question against primary sources only, writes cited findings to `docs/research/<slug>.md`. Auto-triggers when a topic needs investigating. |
| `handoff` | Compacts the current conversation into a handoff document (saved to the OS temp/scratchpad dir) for a fresh agent to continue from, with a "suggested skills" section. User-invoked only (`/handoff`), never auto-triggers. |

## Directory layout

```
~/.claude/skills/
├── brainstorm/SKILL.md
├── spec/SKILL.md
├── implement/SKILL.md
├── commit/SKILL.md
├── domain-modeling/
│   ├── SKILL.md
│   └── references/
│       ├── GLOSSARY-FORMAT.md
│       └── ADR-FORMAT.md
├── to-diagram/
│   ├── SKILL.md
│   └── references/
│       └── {sequence,c4,flowchart,roadmap}.md   (templates)
├── architecture-patterns/
│   ├── SKILL.md                              (router)
│   └── references/
│       └── {frontend-design,api-design,hexagonal-pattern,unity}.md
├── prototype/SKILL.md
├── diagnosing-bugs/
│   ├── SKILL.md
│   └── scripts/
│       └── hitl-loop.template.sh
├── research/SKILL.md
├── handoff/SKILL.md
└── synced/                                   (gitignored — Anthropic built-ins)
```

## Conventions worth knowing

- `disable-model-invocation: true` on `brainstorm`/`spec`/`implement` — they only run when explicitly invoked as `/brainstorm`, `/spec`, `/implement`, never auto-triggered by the model. `domain-modeling` deliberately does **not** carry this flag — that flag blocks even an explicit `Skill`-tool call from another skill, not just model auto-triggering, which would have broken `brainstorm`/`spec`'s ability to invoke it inline.
- Feature state lives on disk under `.scratch/<feature>/`, never in conversation memory — any phase can resume cold.
- `docs/ADRS/` at the project root holds this framework's own architectural decisions (e.g. `0001-per-step-issue-files.md`) — same mechanism `domain-modeling` uses for any project it's invoked in.
- Adding a new house approach: drop a new file under `architecture-patterns/references/` and add a row to its router table once the approach has proven itself on real work.
- Adding a new diagram type: same pattern under `to-diagram/references/`.
