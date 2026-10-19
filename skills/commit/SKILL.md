---
name: commit
description: Commit changes with a keyed message in the present tense. Use when committing work.
argument-hint: "[spec-folder]"
metadata:
  type: process
---

A commit message describes the project's **new state**: its title completes the sentence "If applied, this commit…", in the present tense, third person: _Adds authentication_, _Fixes rounding in invoice totals_.

```
<key>: <Title>

- <detail>
- <detail>
```

- **Key**: the spec's key, such as `FUNC-1234`. The key is optional: without one, the first line is the title alone.
- **Title**: at most 72 characters, capitalised.
- **Details**: one to three bullets, each one line, carrying what the title cannot: the reason, or a consequence a reader of the log needs.

## Process

1. **Find the key.** With a `[spec-folder]` argument, or a spec folder this conversation works in, read the **Key** line of its `spec.md`; `None` means no key. Otherwise, when `.sdd/settings.json` holds a `null` `commitKeyPattern`, there is no key; else ask the user for one, or none. A key must fully match `commitKeyPattern` when one is set; when it does not, ask the user for one that does. Done when you hold a key or know there is none.

2. **Stage.** Commit what is staged. When nothing is staged, stage every change the current task made, path by path. Done when `git diff --staged` holds exactly the task's changes.

3. **Write the message** from `git diff --staged`. Done when the title fits 72 characters and completes "If applied, this commit…", and the details are one to three bullets.

4. **Commit.** Done when the commit exists and you have reported its SHA (`git rev-parse HEAD`).
