# Setup

The `setup` branch of [`brain`](SKILL.md): connect this machine and the current repository to the second brain, laid out by [FORMAT.md](FORMAT.md). It is safe to run again, and meant to run once in each repository: a rerun keeps what is settled and registers the current repository.

1. **Find the brain.** When `~/.alper/brain.json` names a folder holding `BRAIN.md`, show the path and ask whether to keep it. Otherwise ask whether to clone the team's brain (ask for its remote URL) or create a new one, and where the local clone goes, recommending `~/brain`.
   - To clone: `git clone <url> <path>`.
   - To create: `git init <path>`, write `BRAIN.md` from the template below, `repos/.gitkeep` and `shared/.gitkeep`, and commit them as `Creates the second brain`. Then offer to add a remote; given its URL, run `git remote add origin <url>`. The first push waits for the user's first sync.

   Done when the path holds `BRAIN.md` in a git repository.

2. **Record the path.** Write `~/.alper/brain.json` as `{ "path": "<absolute path, forward slashes>" }`, keeping any other fields. Done when the file reads back with that path.

3. **Register the repository.** Outside a git repository, skip this step and the next. Work out the current repository's slug by FORMAT.md and confirm it with the user. Create `repos/<slug>/.gitkeep` when the folder is missing. Add the slug to the **Repositories** list in `BRAIN.md`, with a one-line summary taken from the repository's README. Commit both as `Registers <slug>`. Done when `BRAIN.md` lists the slug and the brain's `git status --porcelain` lists nothing.

4. **Point the agent file.** Ask whether to write the section below into the repository's `AGENTS.md` or `CLAUDE.md` (recommending the one that exists), so every agent here recalls the brain and teammates learn where it lives. Say that the file is committed, so the brain's remote URL becomes visible to whoever can read this repository. Write it once, replacing any earlier copy:

   ```markdown
   ## Second brain

   Team notes on this repository live in the second brain (`<remote URL, or "local only">`), under `repos/<slug>/`. Recall them with the `brain` skill before starting work. Run `/brain setup` once to clone it.
   ```

   Done when the user declines, or the file holds the section once.

5. **Hand over.** Report the brain's path, the slug, and what you committed in which repository. Say that nothing was pushed: `/brain sync` shares the notes, and `/brain graph` draws them. Done when the user has the report.

## BRAIN.md template

```markdown
# Second brain

Our team's notes on our repositories, one fact per file, versioned in git. Notes on one repository live in `repos/<slug>/`, and notes spanning several in `shared/`. They follow [the brain format](https://github.com/alexandrecsp/skills/blob/main/skills/brain/FORMAT.md).

To join, install the plugin (`/plugin marketplace add alexandrecsp/skills`, then `/plugin install alper-skills@alper`) and run `/brain setup`. `/brain graph` draws the notes as a graph. Obsidian can also open this folder as a vault.

## Repositories
```
