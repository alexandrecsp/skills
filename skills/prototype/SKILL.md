---
name: prototype
description: Build a throwaway prototype that answers one design question. Use when the user wants to feel out whether logic or a state model is right, or explore what a UI should look like.
argument-hint: "[question]"
metadata:
  type: process
---

A prototype is **throwaway** code that answers one question. The question decides its shape, and the **verdict** (the answer, with the question it settled) is the only part that ships.

## Pick the branch

Name the question from the user's prompt and the code around it; ask when it is unclear and the user is here.

- **"Does this logic or state model feel right?"** → [LOGIC.md](LOGIC.md): one shareable HTML file with free-play buttons and guided walkthroughs that push a pure state module through the cases that are hard to reason about on paper.
- **"What should this look like?"** → [UI.md](UI.md): several structurally different variants on one route, switched by a `?variant=` search param and a floating bar.

The two branches build very different things, so a wrong pick wastes the prototype. When the question stays ambiguous and the user is away, follow the code around it (a backend module → logic, a page or component → UI) and state that assumption at the top of the prototype.

## Rules for both branches

- **Marked throwaway.** Place it next to the module or page it serves, so its context is obvious, with `prototype` in its file or route name. Follow the project's existing routing and folder conventions.
- **One command to run.** A logic prototype opens by double-click; a UI prototype starts with one command from the project's task runner (`pnpm <name>`, `bun <path>`).
- **In-memory state.** When persistence is itself the question, use a scratch database or local file named so it is obviously disposable (`prototype-wipe-me.db`).
- **Speed over polish.** Write only the code that makes it runnable: tests, error handling and abstractions belong to the real build.
- **State on screen.** After every action (logic) or variant switch (UI), render the full relevant state so the user sees what changed.

## Capture

The prototype is done when its question has a verdict. Then:

1. **Keep the prototype** as a primary source: commit it on a branch `prototype/<slug>`, off the base branch, and leave the base branch free of prototype code.
2. **Record the verdict** where the decision lives: in the spec's Further Notes when a spec under `.sdd/specs/` covers the work, with the branch name; otherwise give it to the user with the branch name, to carry into `/brainstorm` or `/to-spec`.

The validated decision reaches the real code through the pipeline, rebuilt test-first, never by promoting prototype code. Done when the prototype is on its branch and the verdict is recorded.
