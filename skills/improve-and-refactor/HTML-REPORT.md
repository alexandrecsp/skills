# HTML report

One self-contained HTML file. Tailwind and Mermaid come from CDNs; everything else is inline and static. The **diagrams** carry the report: prose stays sparse and plain, and when a diagram needs a paragraph to be understood, redraw the diagram.

The report is a sibling of `.sdd/dashboard.html` and wears its theme: zinc surfaces in light and dark, emerald as the one accent, rounded cards with a gradient accent bar, mono uppercase eyebrows. When this file and the dashboard disagree on a style, the dashboard wins.

## Scaffold

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Refactor review · {{repo name}}</title>
<script src="https://cdn.tailwindcss.com/3.4.16"></script>
<script type="module">
  import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs";
  const dark = matchMedia("(prefers-color-scheme: dark)").matches;
  mermaid.initialize({ startOnLoad: true, theme: dark ? "dark" : "neutral", themeVariables: { fontFamily: "inherit" } });
</script>
<style>
  .dots { background-image: radial-gradient(rgb(161 161 170 / .12) 1px, transparent 1px); background-size: 18px 18px; }
  .seam { stroke-dasharray: 4 4; }
  .leak { stroke: rgb(239 68 68); }
  ::-webkit-scrollbar { width: 8px; height: 8px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: rgb(161 161 170 / .35); border-radius: 9999px; border: 2px solid transparent; background-clip: content-box; }
</style>
</head>
<body class="bg-white text-zinc-900 dark:bg-zinc-950 dark:text-zinc-100 antialiased">
  <header class="sticky top-0 z-20 border-b border-zinc-200 dark:border-zinc-800 bg-white/90 dark:bg-zinc-950/90 backdrop-blur">
    <div class="mx-auto max-w-7xl px-4 h-14 flex items-center gap-3 text-sm">
      <span class="flex items-center gap-2 font-semibold tracking-tight">
        <span class="grid h-7 w-7 place-items-center rounded-lg bg-gradient-to-br from-emerald-400 to-sky-500 text-xs font-bold text-white shadow-sm">S</span>SDD
      </span>
      <span class="text-zinc-300 dark:text-zinc-700">/</span><span class="text-zinc-600 dark:text-zinc-300">Refactor review</span>
      <nav class="ml-auto hidden sm:flex gap-1 font-mono text-xs">...one anchor per candidate: #1, #2, …...</nav>
    </div>
  </header>
  <main class="mx-auto max-w-7xl px-4 py-6 space-y-10">
    <section id="hero">...</section>
    <section id="candidates" class="space-y-8">...</section>
  </main>
</body>
</html>
```

Every surface carries its `dark:` pair, as in the dashboard: `border-zinc-200 dark:border-zinc-800`, `bg-white dark:bg-zinc-900/40`, `text-zinc-500 dark:text-zinc-400`.

## Hero

The dashboard's hero card: `relative overflow-hidden rounded-2xl border bg-gradient-to-br from-white to-zinc-50 dark:from-zinc-900 dark:to-zinc-950 p-6 sm:p-8`, with its two blurred glows (`bg-emerald-400/20` top right, `bg-sky-400/15` bottom left, `blur-3xl`, `pointer-events-none`). Inside:

- **Eyebrow**: repo name and date, `font-mono text-xs uppercase tracking-widest text-zinc-500`.
- **Headline**: `text-3xl font-semibold tracking-tight`, the count ("7 refactor candidates"), and one line under it naming the scanned areas.
- **Stat tiles**: a `dl` of tiles (`rounded-xl border bg-white/70 dark:bg-zinc-900/60 px-4 py-3 backdrop-blur`, value `text-2xl font-semibold tabular-nums`) for Candidates, Strong, Worth exploring, Speculative.
- **Top recommendation**: below a `border-t` divider, under the label "Start here" in the eyebrow style: the candidate to tackle first by number and title, one sentence on why, linking to its card's anchor.
- **Legend**: a row of `text-xs text-zinc-500` swatches: solid box = module, dashed line = seam, red arrow = leak, dark box = deep module.

## Candidate card

One `<article id="c{{n}}">` per candidate, in number order, built like the dashboard's spec card: `relative overflow-hidden rounded-xl border border-zinc-200 dark:border-zinc-800 bg-white dark:bg-zinc-900/40 p-6`, with an `absolute inset-x-0 top-0 h-1 bg-gradient-to-r` accent bar in its strength's gradient.

| Strength          | Accent bar                   | Pill                                                                     |
| ----------------- | ---------------------------- | ------------------------------------------------------------------------ |
| `Strong`          | `from-emerald-400 to-teal-500` | `bg-emerald-100 text-emerald-800 dark:bg-emerald-950 dark:text-emerald-300` |
| `Worth exploring` | `from-amber-400 to-orange-500` | `bg-amber-100 text-amber-800 dark:bg-amber-950 dark:text-amber-300`         |
| `Speculative`     | `from-zinc-300 to-zinc-400 dark:from-zinc-700 dark:to-zinc-600` | `bg-zinc-100 text-zinc-700 dark:bg-zinc-800 dark:text-zinc-300` |

In order:

- **Meta row**: the lens (`deepening` or the smell's name) and each dependency category (`in-process`, `local-substitutable`, `remote but owned`, `true external`) as `rounded bg-zinc-100 dark:bg-zinc-800 px-1.5 py-0.5 font-mono` chips, and the strength pill pushed right (`ml-auto rounded-full px-2 py-0.5 text-xs font-medium`).
- **Title**: the number, then the refactor's name ("#3 Collapse the Order intake pipeline"), `text-xl font-semibold tracking-tight`, the number in `font-mono text-zinc-400`.
- **Files**: a `font-mono text-xs text-zinc-500` list.
- **Before / after diagram**: the centrepiece, two panels side by side (stacked below `md`), each about 320px tall, on a `dots rounded-xl border bg-zinc-50/80 dark:bg-zinc-900/40` canvas with a `text-xs uppercase tracking-widest text-zinc-500` label: "Before", "After".
- **Problem**: one sentence, what hurts.
- **Refactor**: one sentence, what changes.
- **Wins**: bullets of at most six words, each naming its gain in `codebase-design` terms ("locality: bugs concentrate in one module", "leverage: one interface, N call sites", "tests cross one interface"), marked with the dashboard's emerald check.
- **ADR callout**, when the candidate reopens an ADR: one line in an amber box (`rounded-lg border border-amber-300 bg-amber-50 dark:border-amber-900 dark:bg-amber-950/40`), "reopens ADR-0007 because…".

## Diagram patterns

Match the pattern to the candidate, and vary them across the report.

- **Mermaid flowchart or sequence**: for graph-shaped structure, "X calls Y calls Z, look at the mess", or "before: 6 round-trips; after: 1". Colour leaks red and the deep module dark with `classDef`.
- **Hand-built boxes and arrows**: `<div>` modules with inline SVG arrows over a relative container, for an after-diagram that shows one thick-bordered deep module with its internals greyed out. Arrows take the dashboard's graph edges: `stroke-width="1.5"`, `stroke-zinc-500 dark:stroke-zinc-400`, a filled arrowhead marker.
- **Cross-section**: stacked horizontal bands (`h-12 border-l-4`) for the layers a call passes through. Before: six thin pass-through layers; after: one thick band named for its responsibility.
- **Mass diagram**: two rectangles per module, interface beside implementation. Shallow: the interface nearly as tall as the implementation; deep: a short interface over a tall implementation.
- **Call-graph collapse**: before, a tree of calls as nested boxes; after, one box with the now-internal calls faded inside it.

The diagram colours, in light and dark:

- **Module**: `rounded-lg border border-zinc-300 dark:border-zinc-700 bg-white dark:bg-zinc-950 shadow-sm`.
- **Deep module**: `rounded-lg border-2 border-zinc-900 bg-zinc-900 text-white dark:border-zinc-100 dark:bg-zinc-100 dark:text-zinc-900`, the dashboard's primary-button pair.
- **Seam**: a `.seam` dashed line in `zinc-400`.
- **Leak**: a `.leak` arrow in red, and red only there.
- **Faded internals**: `opacity-40`.

Label modules inside diagrams with `text-xs uppercase tracking-wider`, so they read as schematic rather than UI.

## Voice

Architecture words come from the `codebase-design` vocabulary and domain words from `GLOSSARY.md`, used exactly, in short declarative phrases:

- "Order intake module is shallow: interface nearly matches the implementation."
- "Pricing leaks across the seam."
- "Deepen: one interface, one place to test."
- "Two adapters justify the seam: HTTP in production, in-memory in tests."
