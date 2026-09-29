---
name: prototype
description: Settles ONE design question that talking can't resolve — a state model or a UI direction — by building throwaway code and parking it on its own never-merged branch. Two triggers only — typed directly (`/prototype`) for a standalone question, or `implement` invoking it mid-step when a step depends on something `spec` never actually settled — never fires mid-`brainstorm`/`spec`.
---

# Prototype

Settles exactly one unresolved design question — never a whole feature — by building throwaway code you can react to instead of reason about. Standalone: reach for it directly the moment a question resists further discussion. Also machinery: `implement` (phase 3 of `brainstorm` → `spec` → `implement`) invokes it mid-step when a step turns out to depend on a state model or UI direction that `spec` never actually settled.

## The one question

State the question in one sentence before writing anything, and put that sentence at the top of whatever you build. "Does this state model feel right?" or "what should this screen look like?" — not "let's explore the onboarding flow." If you can't compress it to one sentence, it's not one question yet; split it or push back on whoever asked.

## Throwaway is a constraint on how it's written, not a promise to delete it

No tests, no error handling beyond what makes it run, no abstractions, no persistence, no real backend — none of that helps answer the one question. State lives in memory only. The moment you're hardening it (adding a test, wiring the real DB, generalizing for a case you might want later), you've stopped prototyping and started building — stop, and go through `spec` instead.

## Two branches

The question picks which one:

- **"Does this state model / logic feel right?"** → one self-contained HTML file, no build, no server, opens by double-click. A labelled state panel that re-renders after every action, free-play buttons for poking at it in any order, and tabbed guided walkthroughs (one scenario per tab, the ordered buttons to press underneath). Label everything in domain language — this file gets handed to whoever actually knows the domain, not just read by whoever's coding. Keep the logic itself (reducer, machine, plain functions) clean of the DOM in its own module, so the validated version lifts straight into the real code instead of getting rewritten.
- **"What should this look like?"** → several UI variants on one route, switchable from a floating bottom bar and a `?variant=` URL param. Variants must disagree about structure and information hierarchy, not colour — three tweaked card grids is wallpaper, not a prototype. Render against real data and real density wherever you can; a variant judged in a vacuum always looks fine.

## The prototype is a primary source

Two things survive a finished prototype, and they go to different places.

The **prototype code** is evidence, not deliverable. It never merges into main — there's nothing there worth maintaining and it rots the moment the question is settled. Commit it to `feat/<feature-slug>/prototype-<question-slug>`, branched off the feature branch, whenever one exists (invoked by `implement`, or run standalone while a feature branch is already checked out). If no feature branch exists yet, branch `prototype/<question-slug>` off main instead — same rule, just nothing to nest under.

The **answer** goes durable, and always gets at least one line in `design.md`'s Approach or Contracts section — the question, the answer, and the branch name — so whoever reads the design later knows a real artifact backs the claim instead of just conversation. Working standalone with no `design.md` to write into, put that same line wherever the decision actually needs to land (a commit message, the doc or ticket that raised the question).

That pointer records *what* was decided, not *why*. Whether *why* also needs recording is the same call `spec` already makes for any Approach decision: invoke `domain-modeling` for an ADR only if the decision is hard to reverse, would surprise a future reader without context, and came out of a genuine trade-off — the same bar, not a lighter one for prototypes. Needing a prototype at all is weak evidence the question clears that bar already (you don't usually build one for something obvious), so expect to write the ADR more often than not — but it's not automatic. A prototype that resolves a genuinely low-stakes, reversible question just gets the pointer and skips the ADR.

## When invoked by `implement`

A step's `issues/<id>.md` can turn up a question `spec` assumed was settled but wasn't. See `implement/SKILL.md`'s "When a step rests on an unsettled design question" for how that gate dispatches this skill and resumes the step — this file only describes what runs once invoked.

## It's working if

- The question is written down in one sentence, on the artifact itself, not just held in your head.
- Someone who doesn't read code can drive the logic demo unassisted and describe what they saw in their own words.
- Someone reacts with "wait, that shouldn't be possible" or "huh, I assumed X" — that's a bug in the idea, which is the entire point of building this instead of just discussing it.
- UI variant feedback disagrees about structure ("the header from B with the sidebar from C"), not colour or copy.
- It resolves in one sitting. Still building it a day later means the question was too big — split it and answer the pieces separately.
- When it's over, `design.md` (and an ADR, if it cleared the bar) carries the decision, main carries none of the prototype code, and the branch is still there for whoever wants to check.

## Where it fits

Standalone and reach-for-it-anytime: nothing stops you typing `/prototype` outside `brainstorm`/`spec`/`implement` entirely, for any question that talking won't resolve.

Inside the spec-driven workflow, it's `implement`'s escape hatch for exactly the situation `spec` couldn't fully avoid: a step whose behavior turns out to rest on a question nobody actually settled. It never runs during `brainstorm` or `spec` themselves — if a question is this concrete before code exists, it's usually cheaper to keep talking or note the open question in `design.md` than to build something throwaway. Reach for a prototype only once a step is genuinely blocked on it.
