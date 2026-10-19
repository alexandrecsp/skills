# alper-skills

Spec-driven agent skills for Claude Code, packaged as an installable plugin. They take a feature from a loose idea to reviewed, committed code: you brainstorm it, write it down as a spec, cut the spec into tickets, and build the tickets test-first. You make the decisions. The agent does the digging, writing and building.

## Install

```
/plugin marketplace add <path-or-git-url-of-this-repo>
/plugin install alper-skills@alper
```

The skills then show up as `/alper-skills:<name>`. Below they are written by their short names.

## The pipeline

```
/setup-repository          once per repo: creates .sdd/
        │
/brainstorm  ──────────────  a shared understanding: What, then How
        │
/to-spec  ─────────────────  .sdd/specs/NNN-<slug>/spec.md
        │
/to-tickets  ──────────────  milestones/NNN-<milestone>/ with tickets
        │
/build  (once per milestone) ┐
   ├─ per ticket: sub-agent → tdd → commit
   ├─ check the milestone's "Done when"
   └─ review-changes → fix  ┘
```

Each stage writes files the next one reads, and each ends by telling you the next command to run. You can stop between any two stages, or join partway through. For example, `/to-tickets` works on the current conversation without a spec, and `review-changes` works on any branch.

### 0. `/setup-repository`

Prepares the repo. It creates `.sdd/`, which holds everything the pipeline shares and is added to `.gitignore`:

```
.sdd/
├── dashboard.html   ← open in a browser, pick the .sdd folder, see every spec's tickets and status
├── settings.json    ← { "commitKeyPattern": "FUNC-\\d+" } or null
└── specs/
```

`commitKeyPattern` is the regex for an optional key at the start of every commit message, such as a ticket ID. It is safe to run again: a rerun refreshes the dashboard and keeps everything else.

### 1. `/brainstorm [topic]`

The agent interviews you, one round of questions at a time, until you share an understanding of the work. The decisions form a **design tree** with two phases:

- **What** comes first: the problem, scope, behaviour, constraints and what success looks like.
- **How** comes second: the approach, architecture, data and design patterns (via `design-patterns`).

Each round asks every question that can be answered *now*, with options and a recommended answer, so replying `ok` accepts all the recommendations. When a question depends on a fact about the codebase, the agent looks it up instead of asking you. Each phase ends with a summary for you to confirm.

`/brainstorm-with-docs` runs the same interview together with `domain-modeling`. Terms are challenged against `GLOSSARY.md` and written down as they settle, and decisions that are hard to reverse are offered as ADRs in `docs/adr/`.

### 2. `/to-spec`

Turns the conversation into `.sdd/specs/NNN-<slug>/spec.md`. The spec contains only what the conversation settled. Anything left open is recorded as an open question. Before writing, the agent proposes two things for you to confirm:

- the **seams** where tests will prove the feature works, as few as possible
- the **design patterns**, if the brainstorm never weighed them

It also asks for the spec's commit key. The spec covers the problem, solution, user stories and out-of-scope items (the What), then implementation decisions, testing decisions and design patterns (the How).

### 3. `/to-tickets [spec-folder]`

Breaks the spec into ordered **milestones**, and each milestone into **tickets**:

- A milestone is an increment that leaves the code working and tested. It has a check the builder can run.
- A ticket is a **tracer bullet**: a thin vertical slice through every layer it needs, verifiable at a confirmed seam, and small enough for one agent with a fresh context. Prefactoring tickets come first. Wide mechanical refactors follow `WIDE-REFACTOR.md` so each batch stays green.

You review the breakdown (boundaries, granularity, dependencies) until you approve it. Then it writes:

```
.sdd/specs/NNN-<slug>/
├── spec.md
└── milestones/
    └── NNN-<milestone-slug>/
        ├── milestone.md
        └── NNN-<ticket-slug>.md   ← Status: todo | done, Commit: <sha>, Depends on, Notes
```

### 4. `/build [spec-or-milestone-folder]`

Builds **one milestone per run**:

1. It picks the earliest milestone that still has `todo` tickets.
2. For each ticket, in order, it starts a fresh **implementer sub-agent**. That sub-agent reads the ticket, the milestone, the spec, and the notes and commits of earlier tickets. It builds the ticket with `tdd` (red, then green, at the ticket's seams) and commits with `commit`. Then it marks the ticket `done` with its SHA and leaves notes for the next ticket.
3. It runs the full suite and the milestone's **Done when** check.
4. It runs `review-changes` over the milestone's whole diff. Spec findings and hard violations are fixed automatically. You choose which judgement calls to fix.

Run `/build` again for the next milestone. Because progress lives in the ticket files, a build that stops can pick up where it left off.

## Skills used along the way

| Skill | Role |
|---|---|
| `tdd` | Red then green, one vertical slice at a time, with tests only at confirmed seams. |
| `commit` | `<KEY>: <Present-tense title>` plus one to three bullets. Reads the key from the spec. |
| `review-changes` | Reviews a diff in two separate sub-agents. **Standards** checks repo docs, glossary, ADRs and the smell baseline. **Spec** checks for missing requirements, scope creep and requirements built wrong. |
| `domain-modeling` | Maintains `GLOSSARY.md` and ADRs, including multi-context repos with `GLOSSARY-MAP.md`. |
| `design-patterns` | A catalogue of patterns and when to choose each one. Each choice is recorded as Pattern, Where and Why. |
| `codebase-design` | Vocabulary for deep modules: module, interface, seam, adapter, depth. |
| `writing-for-agents` | How to write skills, `CLAUDE.md` and `AGENTS.md`. Used when editing this repo. |

The pipeline skills (`setup-repository`, `brainstorm-with-docs`, `to-spec`, `to-tickets`, `build`) only run when you type their command. The others can also be invoked by the agent when it needs them.

## Repository layout

```
.claude-plugin/   plugin.json and marketplace.json
skills/<name>/    SKILL.md plus any reference files it points to
```
