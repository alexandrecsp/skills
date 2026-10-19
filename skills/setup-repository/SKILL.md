---
name: setup-repository
description: Prepare a repository for the spec workflow, creating the .sdd/ folder, its settings, and the dashboard.
disable-model-invocation: true
metadata:
  type: process
---

Prepare the repository for `to-spec`, `to-tickets`, `build` and `commit`. Everything they share lives in `.sdd/` at the repo root, kept out of git:

```
.sdd/
├── dashboard.html   ← copied from this skill's folder
├── settings.json
└── specs/           ← to-spec and to-tickets write here
```

Running it again is safe: it refreshes the dashboard and keeps everything else.

## Process

1. **Find the root.** Work from `git rev-parse --show-toplevel`. When this is not a git repository, report that and stop. Done when you hold the root.

2. **Settle the commit key.** A commit message can start with a **key**, such as `FUNC-1234` or `INC1234`, and `settings.json` stores the regex every key must fully match, or `null` when this repo's commits carry no key. When `.sdd/settings.json` already holds a `commitKeyPattern`, show it and ask whether to keep it. Otherwise ask the user for an example key, or whether commits go without one; given an example, propose a regex for it (`FUNC-1234` → `FUNC-\d+`), and confirm. Done when the user confirms a pattern that fully matches their example, or confirms there is no key.

3. **Write `.sdd/`.** Create `.sdd/specs/`. When an old `.scratch/specs/` exists, ask the user whether to move its spec folders into `.sdd/specs/`, and move them when they agree. Write `.sdd/settings.json` as `{ "commitKeyPattern": "<regex>" }` (or `null`), keeping any other fields already there. Copy [dashboard.html](dashboard.html) to `.sdd/dashboard.html`, replacing any older copy. Done when all three exist.

4. **Ignore it.** When `git check-ignore -q .sdd/settings.json` fails, append `.sdd/` to the root `.gitignore`, creating the file when needed. Done when that check passes.

5. **Hand over.** Report what you created or changed, how to open the dashboard (open `.sdd/dashboard.html` in a browser, then pick the `.sdd` folder), and the next step: brainstorm a feature, then `/to-spec`. Done when the user has the report.
