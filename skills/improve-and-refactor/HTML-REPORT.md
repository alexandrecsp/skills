# HTML report

Built on the [report theme](../dashboard/THEME.md): its scaffold, hero, card and diagram canvas, filled as below. The **diagrams** carry the report: prose stays sparse and plain, and when a diagram needs a paragraph to be understood, redraw the diagram.

## Scaffold

- **Report name**: "Refactor review".
- **Nav**: one anchor per candidate: `#1`, `#2`, …
- **Sections**: `<section id="candidates" class="space-y-8">`.
- **Own classes**: `.seam { stroke-dasharray: 4 4; }` and `.leak { stroke: rgb(239 68 68); }`.

## Hero

- **Headline**: the count ("7 refactor candidates"), and under it one line naming the scanned areas.
- **Stat tiles**: Candidates, Strong, Worth exploring, Speculative.
- **Start here** row: the candidate to tackle first by number and title, one sentence on why, linking to its card's anchor.
- **Legend** row: `text-xs text-zinc-500` swatches: solid box = module, dashed line = seam, red arrow = leak, dark box = deep module.

## Candidate card

One `<article id="c{{n}}">` card per candidate, in number order, its accent bar in its strength's gradient:

| Strength          | Accent bar                   | Pill                                                                     |
| ----------------- | ---------------------------- | ------------------------------------------------------------------------ |
| `Strong`          | `from-emerald-400 to-teal-500` | `bg-emerald-100 text-emerald-800 dark:bg-emerald-950 dark:text-emerald-300` |
| `Worth exploring` | `from-amber-400 to-orange-500` | `bg-amber-100 text-amber-800 dark:bg-amber-950 dark:text-amber-300`         |
| `Speculative`     | `from-zinc-300 to-zinc-400 dark:from-zinc-700 dark:to-zinc-600` | `bg-zinc-100 text-zinc-700 dark:bg-zinc-800 dark:text-zinc-300` |

In order:

- **Meta row**: the lens (`deepening` or the smell's name) and each dependency category (`in-process`, `local-substitutable`, `remote but owned`, `true external`) as chips, and the strength pill.
- **Title**: the number, then the refactor's name ("#3 Collapse the Order intake pipeline").
- **Files**.
- **Before / after diagram**: the centrepiece, two panels side by side (stacked below `md`), each about 320px tall, each on a diagram canvas labelled "Before", "After".
- **Problem**: one sentence, what hurts.
- **Refactor**: one sentence, what changes.
- **Wins**: check bullets of at most six words, each naming its gain in `codebase-design` terms ("locality: bugs concentrate in one module", "leverage: one interface, N call sites", "tests cross one interface").
- **ADR callout**, when the candidate reopens an ADR: "reopens ADR-0007 because…".

## Diagram patterns

Match the pattern to the candidate, and vary them across the report.

- **Mermaid flowchart or sequence**: for graph-shaped structure, "X calls Y calls Z, look at the mess", or "before: 6 round-trips; after: 1". Colour leaks red and the deep module dark with `classDef`.
- **Hand-built boxes and arrows**: `<div>` modules with inline SVG arrows over a relative container, for an after-diagram that shows one thick-bordered deep module with its internals greyed out.
- **Cross-section**: stacked horizontal bands (`h-12 border-l-4`) for the layers a call passes through. Before: six thin pass-through layers; after: one thick band named for its responsibility.
- **Mass diagram**: two rectangles per module, interface beside implementation. Shallow: the interface nearly as tall as the implementation; deep: a short interface over a tall implementation.
- **Call-graph collapse**: before, a tree of calls as nested boxes; after, one box with the now-internal calls faded inside it.

Beside the theme's module and arrow, in light and dark:

- **Deep module**: `rounded-lg border-2 border-zinc-900 bg-zinc-900 text-white dark:border-zinc-100 dark:bg-zinc-100 dark:text-zinc-900`, the dashboard's primary-button pair.
- **Seam**: a `.seam` dashed line in `zinc-400`.
- **Leak**: a `.leak` arrow in red, and red only there.
- **Faded internals**: `opacity-40`.

## Voice

Architecture words come from the `codebase-design` vocabulary and domain words from `GLOSSARY.md`, used exactly, in short declarative phrases:

- "Order intake module is shallow: interface nearly matches the implementation."
- "Pricing leaks across the seam."
- "Deepen: one interface, one place to test."
- "Two adapters justify the seam: HTTP in production, in-memory in tests."
