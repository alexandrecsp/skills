---
name: commit
description: Split the working-tree changes into green commits (each one leaves the app building and its tests passing) and write each message as a verb-first headline stating the application's new state after applying it. Use when committing changes, writing a commit message, or when a workflow says to commit.
---

Turn the working tree into a short series of **green** commits. Green means the application builds and its tests pass at that commit, so any commit can be checked out, reverted, or bisected on its own.

## Process

### 1. Read the changes

Run `git status`, `git diff` and `git diff --staged`. Read `git log --oneline -n 10` for the repo's message language and style. Write nothing from the user's description alone while the diff is available.

Done when every changed file and hunk is accounted for.

### 2. Group into commits

A group is one **effect**: one thing that is true after the commit and wasn't before. Order the groups so each stands on the ones before it:

- Prefactoring comes before the feature it enables.
- Tests travel with the code they cover.
- A rename, a move or a formatting change is its own group, apart from behaviour changes.
- Groups that only work together (a signature change and its callers) merge into one; green outranks small.

Stage each group by explicit path (`git add <paths>`). When hunks of one file belong to different groups, stage them non-interactively: trim a patch from `git diff <file>` and apply it with `git apply --cached`.

Done when the groups cover the whole diff and one sentence names each group's effect.

### 3. Verify green, then commit, per group

For each group, in order:

1. Stage the group. Set the rest aside so the checks see only this commit (`git stash push --keep-index --include-untracked`).
2. Run the repo's fast checks: typecheck, build, and the tests the group touches. Take the commands from the repo (`package.json` scripts, Makefile, CI config).
3. Restore the rest (`git stash pop`).
4. Write the message (below), then commit.

A red group merges into its dependency and is checked again. Checks that cannot run here are reported to the user with the commit, never skipped silently.

Done when every commit was checked green and `git status` shows nothing left over, or only what the user chose to keep out.

### 4. Approve

Show the messages before committing. When a workflow invoked this skill with no one there to approve mid-run (`implement`'s commit step), that invocation is the approval: commit, and include each message verbatim in the report.

## The message

The message answers one question: **after applying this commit, what is the new state of the application?**

**Headline**: one line, about 70 characters at most, that starts with a verb completing *"Ao aplicar este commit, ele…"* (*"If applied, this commit will…"*), and names the new state. Write the completion only, verb first, as an action the commit performs: the verb form that fits the sentence is third person present in Portuguese (`Altera`, `Corrige`, `Adiciona`) and imperative in English (`Change`, `Fix`, `Add`).

| Instead of | Write |
|---|---|
| `Alteração de nome da variável X` | `Altera o nome da variável X para Y` |
| `Corrigido bug no login` | `Corrige o login para aceitar e-mails com maiúsculas` |
| `Fixed race condition` | `Fix race condition in session cleanup` |

Name what is now true of the application (the behaviour, the name, the rule), not the work that produced it.

**Body**: optional. A blank line, then at most 5 bullets, most significant first, each one a concrete change. Fewer bullets are fine; the headline alone is fine when it says everything.

**Language**: the language of the repo's `git log`; with no history, the language the user writes in.

```
Corrige a limpeza de sessões para não liberar a mesma sessão duas vezes

- Protege a remoção de sessões com um lock para requisições concorrentes
- Adiciona teste de regressão que reproduz a intercalação que causava o crash
- Registra em debug os IDs das sessões removidas
```
