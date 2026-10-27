---
name: setup-repository
description: Prepare a repository for the spec workflow, creating the .sdd/ folder and its settings.
disable-model-invocation: true
metadata:
  type: process
---

Prepare the repository for `to-spec`, `to-tickets`, `build` and `commit`. Everything they share lives in `.sdd/` at the repo root, kept out of git:

```
.sdd/
├── settings.json
└── specs/           ← to-spec and to-tickets write here
```

It also settles the repo's layout in `docs/STRUCTURE.md`, and points the repo's `AGENTS.md` or `CLAUDE.md` at the framework's files. Both are committed: they are team standards, not pipeline state.

Running it again is safe: it keeps everything already settled unless the user changes it.

## Process

1. **Find the root.** Work from `git rev-parse --show-toplevel`. When this is not a git repository, report that and stop. Done when you hold the root.

2. **Settle the commit key.** A commit message can start with a **key**, such as `FUNC-1234` or `INC1234`, and `settings.json` stores the regex every key must fully match, or `null` when this repo's commits carry no key. When `.sdd/settings.json` already holds a `commitKeyPattern`, show it and ask whether to keep it. Otherwise ask the user for an example key, or whether commits go without one; given an example, propose a regex for it (`FUNC-1234` → `FUNC-\d+`), and confirm. Done when the user confirms a pattern that fully matches their example, or confirms there is no key.

3. **Write `.sdd/`.** Create `.sdd/specs/`. When an old `.scratch/specs/` exists, ask the user whether to move its spec folders into `.sdd/specs/`, and move them when they agree. Write `.sdd/settings.json` as `{ "commitKeyPattern": "<regex>" }` (or `null`), keeping any other fields already there. When an old `.sdd/dashboard.html` exists, delete it: the `dashboard` skill now serves one page for every repo. Done when `.sdd/specs/` and `.sdd/settings.json` exist and `.sdd/dashboard.html` does not.

4. **Ignore it.** When `git check-ignore -q .sdd/settings.json` fails, ask the user whether to append `.sdd/` to the root `.gitignore` (creating the file when needed), recommending it. When the user declines, or the repo's `.gitignore` is off-limits, append `.sdd/` to `.git/info/exclude` instead: it ignores the folder locally, with nothing to commit. When `git ls-files .sdd` lists files already committed, show them and, when the user agrees, untrack them with `git rm -r --cached .sdd`. Done when the check passes and `git ls-files .sdd` lists nothing.

5. **Settle the layout.** Invoke the `project-structure` skill. When `docs/STRUCTURE.md` exists, show its archetype and ask whether to keep it. Otherwise detect the archetype of each app, then propose `docs/STRUCTURE.md`: the layout the code already uses, or, in a repo without code, the archetype's layout; iterate until the user confirms it, then write it. Done when the user confirms `docs/STRUCTURE.md`.

6. **Settle the suites.** The `tdd` skill runs two suites: the **unit suite** (the linter, the typechecker, the unit and integration tests) and the **e2e suite** (the tests that drive a running app: a browser, a device or a deployed service). When the agent file already holds a **Suites** line, show it and ask whether to keep it. Otherwise read the project's scripts and propose one command for each suite, or `none` when the repo has no e2e tests. When one script runs both suites, propose scripts that split them, and add them when the user agrees. Done when the user confirms both commands, and running each one starts the tests it names.

7. **Point the agent file.** Ask whether the repo's agent instructions live in `AGENTS.md` or `CLAUDE.md`, recommending the one that already exists. When the user picks `AGENTS.md` and no `CLAUDE.md` exists, offer a `CLAUDE.md` holding only `@AGENTS.md`: Claude Code reads `CLAUDE.md`, never `AGENTS.md`. In the chosen file, write this section, replacing it when it already exists and creating the file when needed:

   ```markdown
   ## Spec workflow

   - `docs/STRUCTURE.md`: the folder layout and file names. Read it before creating, moving or naming a file.
   - `GLOSSARY.md` and `docs/adr/` (or `GLOSSARY-MAP.md`), when present: the domain's terms and the decisions already made. Name code in the glossary's terms, and work within the ADRs.
   - `.sdd/specs/`: specs, milestones and tickets, kept out of git. On an `sdd/<spec-folder>/spec` or `sdd/<spec-folder>/M<NNN>` branch, that spec folder, and its milestone numbered `NNN`, hold the work's requirements.
   - Suites: unit `<unit command>`; e2e `<e2e command or none>`. The `tdd` skill says when to run each.
   ```

   Done when the chosen file holds the section once, with step 6's commands.

8. **Hand over.** Report what you created, changed or deleted, including the files to commit (`docs/STRUCTURE.md` and the agent file), how to see this repo on the dashboard (run `/dashboard`, then **Add repo** and pick this repo's root, when it is not listed yet), and the next step: brainstorm a feature, then `/to-spec`. Done when the user has the report.
