---
name: commit
description: Write commits whose message states the application's new state, under a gitmoji headline. Use when committing changes, writing a commit message, or when a workflow says to commit.
---

A commit message answers one question: **after applying this commit, what is the new state of the application?**

## Process

### 1. Read the changes

Run `git status`, `git diff` and `git diff --staged`. Run `git log --oneline -n 10` for the repo's message language and style.

Done when every changed file and hunk is accounted for.

### 2. Group into commits

A group is one **effect**: one thing true after the commit that wasn't before. Each commit leaves the build and tests green, so any commit can be reverted or bisected alone.

- Prefactoring lands before the feature it enables.
- Tests travel with the code they cover.
- A rename, move or formatting pass is its own group.
- Changes that only work together (a signature and its callers) are one group.

Stage each group by explicit path (`git add <paths>`); split hunks of one file with `git apply --cached` on a trimmed patch.

Done when one sentence names each group's effect, and the groups cover the whole diff.

### 3. Write the message and commit

Per group, in order: write the message below, run the repo's fast checks (take the commands from `package.json`, Makefile or CI config), then commit. Report checks that cannot run here with the commit.

Show the messages before committing. When a workflow invoked this skill with no one there to approve, that invocation is the approval: commit, and include each message verbatim in the report.

Done when `git status` shows nothing left over, or only what the user chose to keep out.

## The message

```
<gitmoji> <Verb> <new state>

- <concrete change>
```

**Headline**: one line, about 70 characters, opening with the gitmoji that fits the effect, then a verb completing *"If applied, this commit will…"* and naming what is now true of the application (the behaviour, the name, the rule), where the work that produced it stays out of the line. Imperative in English (`Fix`, `Add`); third person present in Portuguese (`Corrige`, `Adiciona`).

| Instead of | Write |
|---|---|
| `Fixed bug in login` | `🐛 Fix login to accept uppercase e-mails` |
| `Variable rename` | `♻️ Rename variable X to Y` |
| `Changes to session cleanup` | `🐛 Fix session cleanup releasing the same session twice` |

**Body**: optional. A blank line, then at most 5 bullets, most significant first, each a concrete change to the application's state. The headline alone is fine when it says everything.

**Language**: the language of the repo's `git log`; with no history, the language the user writes in.

### Gitmoji

One gitmoji per commit, the one naming its effect:

| | Effect |
|---|---|
| ✨ | New feature |
| 🐛 | Bug fix |
| 🚑️ | Critical hotfix |
| ♻️ | Refactor, same behaviour |
| ⚡️ | Performance |
| 💄 | UI and styling |
| ✅ | Tests added or updated |
| 📝 | Documentation |
| 🔧 | Configuration |
| 📦️ | Build, dependencies |
| 👷 | CI |
| 🔥 | Code or files removed |
| 🚚 | Moved or renamed |
| 🎨 | Structure or formatting |
| 🔒️ | Security |
| 💥 | Breaking change |
| 🗃️ | Database |

Anything outside the table: pick from [gitmoji.dev](https://gitmoji.dev).

```
🐛 Fix session cleanup releasing the same session twice

- Guard session removal with a lock for concurrent requests
- Add a regression test reproducing the interleaving that crashed
- Log removed session IDs at debug level
```
