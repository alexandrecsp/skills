# Personal Claude Code Skills

A personal skills framework for [Claude Code](https://claude.com/claude-code), lives at `~/.claude/skills` and applies across all projects. It is a customised fork of [mattpocock/skills](https://github.com/mattpocock/skills): the same small, composable shape, plus house architecture patterns wired through the whole chain.

`synced/` is excluded (see `.gitignore`). It holds Anthropic's built-in skills (docs, pdf, pptx, xlsx, morning, skill-creator, import-memory), managed separately.

## The workflow

```
/setup (once per repo)

/brainstorm  →  /to-spec  →  /implement
 or                          or /implement --parallel
/brainstorm-with-docs
```

State lives on disk as markdown under `.scratch/<feature-slug>/`, never in conversation memory, so any step can resume cold:

```
.scratch/<feature-slug>/
├── spec.md
└── issues/
    ├── 01-<slug>.md      Status / Blocked by / Pattern header + acceptance criteria
    └── 02-<slug>.md
```

The ticket format is defined once, in the repo's `docs/agents/issue-tracker.md` (seeded by `/setup`). Statuses: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `claimed`, `resolved`, `wontfix`.

### User-invoked skills (`disable-model-invocation: true`)

| Skill | Purpose |
|---|---|
| `setup` | Once per repo: writes `docs/agents/` (issue tracker, triage labels, domain-doc layout) and an `## Agent skills` block in `CLAUDE.md`/`AGENTS.md`. |
| `brainstorm` | Thin wrapper: calls `brainstorming`. |
| `brainstorm-with-docs` | Calls `brainstorming` and `domain-modeling`, so `CONTEXT.md` terms and ADRs are written as they crystallise. |
| `to-spec` | Two phases, one checkpoint. Agrees test seams and the architecture pattern(s) with the user, then without further approval synthesises the conversation into `spec.md` (no interview) and splits it into tracer-bullet tickets, one file each, with `Blocked by:` edges and a `Pattern:` line. Wide refactors use expand–contract. Pass an existing spec path to start at the tickets. |
| `implement` | Builds a spec or tickets on branch `feat/<spec-name>`. With `--parallel`, orchestrates the whole spec instead: runs the ticket frontier as concurrent subagents in worktrees, merges each into an integration branch, then one review at the end. Otherwise, per ticket: claim, do the work, tick criteria, mark `resolved`. Uses `tdd`, follows the ticket's pattern, reviews with `code-review`, commits with `commit`. Resumable from ticket state. |
| `triage` | Moves issues through the triage state machine and writes agent-ready briefs. |
| `wayfinder` | Plans work too big for one session as a map of decision tickets, resolved one at a time. |
| `improve-codebase-architecture` | Finds deepening opportunities, presents an HTML report, then brainstorms the one you pick. |
| `handoff` | Compacts the conversation into a document a fresh agent can continue from. |

### Model-invoked skills

| Skill | Purpose |
|---|---|
| `brainstorming` | The interview loop: asks every unblocked question of the design tree per round, each with a recommended answer, and sends sub-agents to look up facts. |
| `domain-modeling` | Builds `CONTEXT.md` and ADRs (`docs/adr/`) while designing. |
| `architecture-patterns` | Router to house patterns: hexagonal, API design, frontend UI, Unity. Consulted by `to-spec`, `implement` and review. |
| `codebase-design` | Deep-module vocabulary: interfaces, seams, deepening. |
| `tdd` | Red-green-refactor, with reference on tests and mocking. |
| `diagnosing-bugs` | Feedback-loop-first debugging for hard bugs and regressions. |
| `code-review` | Two-axis review (Standards and Spec) run in parallel sub-agents. |
| `commit` | Splits changes into green commits with verb-first headlines. |
| `prototype` | Throwaway code (logic or UI) to answer one design question. |
| `research` | Background agent that investigates primary sources and writes cited findings. |
| `resolving-merge-conflicts` | Resolves an in-progress merge or rebase from the changes' original intent. |
| `writing-for-agents` | Standards for writing skills, `AGENTS.md` and `CLAUDE.md`. |

## Directory layout

```
~/.claude/skills/
├── setup/  (+ references/)
├── brainstorm/  brainstorm-with-docs/  brainstorming/
├── to-spec/  (+ references/)
├── implement/  (+ references/)
├── triage/  (+ references/)   wayfinder/
├── tdd/  (+ references/)      diagnosing-bugs/  (+ scripts/)
├── code-review/  commit/  resolving-merge-conflicts/
├── domain-modeling/  (+ references/)
├── architecture-patterns/  (router + references/)
├── codebase-design/  improve-codebase-architecture/  (+ references/)
├── prototype/  (+ references/)   research/   handoff/
├── writing-for-agents/  (+ references/)
└── synced/   (gitignored: Anthropic built-ins)
```

## Conventions

- Skills that start a workflow are user-invoked only. Skills other skills call (`brainstorming`, `domain-modeling`, `tdd`, `commit`, ...) carry no such flag, because the flag also blocks calls from other skills through the Skill tool.
- A `Pattern:` line on a spec or ticket names the architecture pattern to follow; `implement` only calls `architecture-patterns` to judge when none is recorded.
- Adding a house approach: drop a file under `architecture-patterns/references/` and add a row to the router table once it has proven itself on real work.
- Follow `writing-for-agents` when creating or editing any skill.
