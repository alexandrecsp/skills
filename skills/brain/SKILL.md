---
name: brain
description: Recall from and record to the second brain, the team's versioned notes on every repository. Use when an agent file names a second brain, when the user asks to remember, recall, sync, set up or graph it, or when you learn a gotcha, decision or contract the repository itself does not record.
argument-hint: "[setup | graph | sync | recall | remember <fact>]"
metadata:
  type: process
---

The **second brain** is a git repository of notes on many repositories, cloned on each teammate's machine and shared by pull and push. [FORMAT.md](FORMAT.md) gives its layout, the repo slug and the note format. It holds what a repository cannot tell by looking: the reason behind a choice, a trap, a contract between two repositories. What the code, git history or docs already say stays there, and the user's personal preferences belong in your own memory.

The first argument picks the branch: `setup` follows [SETUP.md](SETUP.md), `graph` follows [GRAPH.md](GRAPH.md), `recall` follows [RECALL.md](RECALL.md), and `sync` and `remember` run their sections below. With no argument, recall before working in a registered repository, or record when you have learned something the brain should hold.

Every other branch reads the brain's path from `~/.alper/brain.json`. When the user asked for the brain and the file is missing, or the repository is not registered (by [FORMAT.md](FORMAT.md)), tell them to run `/brain setup`; otherwise carry on without it. Run every git command in the brain as `git -C <path>`.

## Record

When you learn something the brain should hold, the user asks you to remember it (`remember <fact>`), or another skill hands you candidates:

1. **Find its home.** Grep the brain's descriptions for a note already covering it. Done when you know whether to update that note or write a new one.
2. **Propose.** Show the user the note's path and full text by FORMAT.md, or the diff to the existing note. It goes in `repos/<slug>/`, or in `shared/` when it spans repositories, linked to its related notes. Several candidates go out as one numbered list, each with its path and text, for the user to pick by number. Done when the user confirms or declines each; revise until they do.
3. **Commit** each confirmed note alone, with a present-tense title: `Records <name> for <slug>`, `Updates <name>` or `Deletes <name>`. Done when the brain's `git status --porcelain` lists nothing.

A wrong or stale note goes the same way: propose deleting it, then commit the deletion. Recording never pushes.

## Sync

Only when the user asks, since a push publishes the notes to every teammate:

1. **Pull** with `git pull --rebase`. A conflict means two people changed one note: merge both versions' facts into one, show the user the merged note, and continue the rebase once they confirm. Done when the rebase completes.
2. **Push.** Show the commits `git log --oneline @{u}..` lists, then `git push`. Done when the push succeeds, or the user has its error.
