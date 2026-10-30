# Report theme

The look shared by every HTML report a skill writes into `.sdd/`: the spec dashboard's, [dashboard.html](dashboard.html). Zinc surfaces in light and dark, emerald as the one accent, rounded cards with a gradient accent bar, mono uppercase eyebrows. When this file and the dashboard disagree on a style, the dashboard wins.

A report is one self-contained file. Tailwind and Mermaid come from CDNs; everything else is inline and static. The report's own reference fills each `{{slot}}` below.

## Scaffold

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{{report name}} · {{repo name}}</title>
<script src="https://cdn.tailwindcss.com/3.4.16"></script>
<script type="module">
  import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs";
  const dark = matchMedia("(prefers-color-scheme: dark)").matches;
  mermaid.initialize({ startOnLoad: true, theme: dark ? "dark" : "neutral", themeVariables: { fontFamily: "inherit" } });
</script>
<style>
  .dots { background-image: radial-gradient(rgb(161 161 170 / .12) 1px, transparent 1px); background-size: 18px 18px; }
  ::-webkit-scrollbar { width: 8px; height: 8px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: rgb(161 161 170 / .35); border-radius: 9999px; border: 2px solid transparent; background-clip: content-box; }
  {{the report's own classes}}
</style>
</head>
<body class="bg-white text-zinc-900 dark:bg-zinc-950 dark:text-zinc-100 antialiased">
  <header class="sticky top-0 z-20 border-b border-zinc-200 dark:border-zinc-800 bg-white/90 dark:bg-zinc-950/90 backdrop-blur">
    <div class="mx-auto max-w-7xl px-4 h-14 flex items-center gap-3 text-sm">
      <span class="flex items-center gap-2 font-semibold tracking-tight">
        <span class="grid h-7 w-7 place-items-center rounded-lg bg-gradient-to-br from-emerald-400 to-sky-500 text-xs font-bold text-white shadow-sm">S</span>SDD
      </span>
      <span class="text-zinc-300 dark:text-zinc-700">/</span><span class="text-zinc-600 dark:text-zinc-300">{{report name}}</span>
      <nav class="ml-auto hidden sm:flex gap-1 font-mono text-xs">{{one anchor per section or item}}</nav>
    </div>
  </header>
  <main class="mx-auto max-w-7xl px-4 py-6 space-y-10">
    <section id="hero">...</section>
    {{the report's sections}}
  </main>
</body>
</html>
```

Every surface carries its `dark:` pair, as in the dashboard: `border-zinc-200 dark:border-zinc-800`, `bg-white dark:bg-zinc-900/40`, `text-zinc-500 dark:text-zinc-400`.

## Hero

The dashboard's hero card: `relative overflow-hidden rounded-2xl border bg-gradient-to-br from-white to-zinc-50 dark:from-zinc-900 dark:to-zinc-950 p-6 sm:p-8`, with its two blurred glows (`bg-emerald-400/20` top right, `bg-sky-400/15` bottom left, `blur-3xl`, `pointer-events-none`). Inside, in order:

- **Eyebrow**: repo name and date, `font-mono text-xs uppercase tracking-widest text-zinc-500`.
- **Headline**: `text-3xl font-semibold tracking-tight`, with one line under it.
- **Stat tiles**: a `dl` of tiles (`rounded-xl border bg-white/70 dark:bg-zinc-900/60 px-4 py-3 backdrop-blur`, value `text-2xl font-semibold tabular-nums`).
- **Rows**: below a `border-t` divider, the report's own rows, each under a label in the eyebrow style.

## Card

One `<article>` per item, built like the dashboard's spec card: `relative overflow-hidden rounded-xl border border-zinc-200 dark:border-zinc-800 bg-white dark:bg-zinc-900/40 p-6`, with an `absolute inset-x-0 top-0 h-1 bg-gradient-to-r` accent bar in the gradient the report names. Its parts:

- **Meta row**: chips (`rounded bg-zinc-100 dark:bg-zinc-800 px-1.5 py-0.5 font-mono`), and a pill pushed right (`ml-auto rounded-full px-2 py-0.5 text-xs font-medium`).
- **Title**: `text-xl font-semibold tracking-tight`, a leading handle in `font-mono text-zinc-400`.
- **Files**: a `font-mono text-xs text-zinc-500` list.
- **Checks**: bullets marked with the dashboard's emerald check.
- **Callout**: one line in an amber box (`rounded-lg border border-amber-300 bg-amber-50 dark:border-amber-900 dark:bg-amber-950/40`).

## Diagrams

A diagram sits on a canvas, `dots rounded-xl border bg-zinc-50/80 dark:bg-zinc-900/40`, under a `text-xs uppercase tracking-widest text-zinc-500` label. A hand-built diagram draws:

- **Module**: `rounded-lg border border-zinc-300 dark:border-zinc-700 bg-white dark:bg-zinc-950 shadow-sm`.
- **Arrow**: the dashboard's graph edges, inline SVG with `stroke-width="1.5"`, `stroke-zinc-500 dark:stroke-zinc-400`, and a filled arrowhead marker.

Label modules inside diagrams with `text-xs uppercase tracking-wider`, so they read as schematic rather than UI.
