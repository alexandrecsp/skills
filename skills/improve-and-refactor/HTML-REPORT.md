# HTML report

One self-contained HTML file. Tailwind and Mermaid come from CDNs; everything else is inline and static. The **diagrams** carry the report: prose stays sparse and plain, and when a diagram needs a paragraph to be understood, redraw the diagram.

## Scaffold

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Refactor review for {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral" });
    </script>
    <style>
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## Header

Repo name, date, the scanned areas, and a compact legend: solid box = module, dashed line = seam, red arrow = leak, thick dark box = deep module. The candidates follow directly.

## Candidate card

One `<article>` per candidate, strongest first:

- **Title**: names the refactor ("Collapse the Order intake pipeline").
- **Badges**: strength (`Strong` emerald, `Worth exploring` amber, `Speculative` slate), the lens (`deepening` or the smell's name), and each dependency category (`in-process`, `local-substitutable`, `remote but owned`, `true external`).
- **Files**: a `font-mono text-sm` list.
- **Before / after diagram**: the centrepiece, two columns side by side, about 320px tall.
- **Problem**: one sentence, what hurts.
- **Refactor**: one sentence, what changes.
- **Wins**: bullets of at most six words, each naming its gain in `codebase-design` terms ("locality: bugs concentrate in one module", "leverage: one interface, N call sites", "tests cross one interface").
- **ADR callout**, when the candidate reopens an ADR: one line in an amber box ("reopens ADR-0007 because…").

## Diagram patterns

Match the pattern to the candidate, and vary them across the report.

- **Mermaid flowchart or sequence**: for graph-shaped structure, "X calls Y calls Z, look at the mess", or "before: 6 round-trips; after: 1". Wrap it in a white bordered card, and colour leaks red and the deep module dark with `classDef`.
- **Hand-built boxes and arrows**: `<div>` modules with inline SVG arrows over a relative container, for an after-diagram that shows one thick-bordered deep module with its internals greyed out.
- **Cross-section**: stacked horizontal bands (`h-12 border-l-4`) for the layers a call passes through. Before: six thin pass-through layers; after: one thick band named for its responsibility.
- **Mass diagram**: two rectangles per module, interface beside implementation. Shallow: the interface nearly as tall as the implementation; deep: a short interface over a tall implementation.
- **Call-graph collapse**: before, a tree of calls as nested boxes; after, one box with the now-internal calls faded inside it.

Label modules inside diagrams with `text-xs uppercase tracking-wider`, so they read as schematic rather than UI.

## Style

Editorial: generous whitespace, optional `font-serif` headings, one accent (emerald or indigo), red only for leaks and amber only for warnings.

## Top recommendation

One larger card: the candidate to tackle first, one sentence on why, and an anchor link to its card.

## Voice

Architecture words come from the `codebase-design` vocabulary and domain words from `GLOSSARY.md`, used exactly, in short declarative phrases:

- "Order intake module is shallow: interface nearly matches the implementation."
- "Pricing leaks across the seam."
- "Deepen: one interface, one place to test."
- "Two adapters justify the seam: HTTP in production, in-memory in tests."
