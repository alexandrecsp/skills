---
name: onboard
description: Explore the whole codebase in parallel sub-agents and write an onboarding tour of it as an HTML page, then offer to record the terms and decisions it found undocumented.
disable-model-invocation: true
metadata:
  type: process
---

Write a **tour** of the codebase for a **newcomer**: someone who knows the stack but not this repo. The tour is a guided path of **stops**, from the big picture down to where each thing lives, built on the **journeys** a request takes through the code. Every fact in it is **grounded**: it names the file it comes from, so the newcomer can go and look.

This needs `.sdd/settings.json`; when it is missing, tell the user to run `/setup-repository` and stop.

## Process

1. **Survey.** Read the README, the manifests (`package.json`, `pyproject.toml`, `go.mod`, …), `docs/STRUCTURE.md`, the **Spec workflow** section of `AGENTS.md` or `CLAUDE.md`, and the [domain docs](../domain-modeling/DOMAIN-DOCS.md) with every context's glossary and ADRs. List the tracked files with `git ls-files`. Split the source into **areas**: one per context when `GLOSSARY-MAP.md` names contexts, else one per app in `docs/STRUCTURE.md`, else one per top-level source folder. Split an area holding more than about 150 source files by its next folder level. Tests belong to the area they test; generated and vendored code belongs to none. Done when every tracked source file outside generated and vendored code belongs to exactly one area.

2. **Fan out.** Dispatch one `alper-skills:scout` agent per area, all in one message so they run in parallel. Give each its area's paths, the area list, the glossary, each ADR's title and decision, and its app's section of `docs/STRUCTURE.md` (it has no other access to them), and this brief:

   > Walk this area as a guide preparing a tour of it for a newcomer who knows the stack but not this repo. Report each fact with the file it comes from:
   >
   > - **Purpose**: what the area does, in one sentence in the glossary's terms.
   > - **Entry points**: every route, command, handler, job, page or exported function that the outside world or another area calls, with its kind.
   > - **Key modules**: at most eight, each with its interface in one line.
   > - **Dependencies**: every other area and external system it calls, and how (import, HTTP, queue, database).
   > - **Data**: the entities it owns and where they are stored.
   > - **Tests**: where they live, their kinds, and the command that runs only them.
   > - **Changes**: where a newcomer goes for the area's common changes, each as "to change X, go to `path`".
   > - **Gaps**: terms the code uses that the glossary lacks or defines differently, each with the definition the code implies; and decisions the code makes that are hard to reverse and surprising, but that no ADR records.
   >
   > Report every field, writing "none" for an empty one.

   Done when every area's report has every field, re-dispatching a scout whose report is missing one.

3. **Trace journeys.** Rank the entry points across all reports by how many areas their path crosses, read from the dependencies. Pick the top two or three, preferring different kinds (a user request and a background job over two requests). Dispatch one scout per journey in parallel, with the area reports as its map, and this brief:

   > Trace this entry point end to end, from where it is called to its effect: data stored, response returned, event emitted. Report every **hop** in order, each with its function, its file and its area.

   Done when every journey is an ordered list of hops, each with its function, file and area.

4. **Write the tour.** Write `.sdd/tours/<YYYY-MM-DD-HHmm>.html`, kept with the repo's other pipeline state and out of git, by [TOUR-HTML.md](TOUR-HTML.md). Stamp it with `git rev-parse --short HEAD`, adding "uncommitted changes" when `git status --porcelain` lists any. Open it (`start`, `open` or `xdg-open`) and give the user its absolute path. Done when every area and journey has its stop, and every fact on the page names its file.

5. **Offer the gaps.** Merge the gaps from all reports, one entry per term or decision. List them in your reply, terms first with the definition the code implies, then decisions, and ask which to record. Invoke the `domain-modeling` skill and write each chosen term into its context's `GLOSSARY.md`. A chosen decision becomes an ADR once the user gives its reason: the code shows what was decided, and only the user knows why. Done when the user has chosen or declined every gap, and each chosen one is written.
