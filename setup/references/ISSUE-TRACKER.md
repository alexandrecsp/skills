# Issue tracker

Issues and specs for this repo live as markdown files in `.scratch/`. This file is the single definition of the ticket format; every skill that reads or writes tickets follows it.

## Conventions

- One feature per directory: `.scratch/<feature-slug>/`
- The spec is `.scratch/<feature-slug>/spec.md`, opening with a `Status:` line
- Implementation tickets are one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01`, never a single combined tickets file
- A ticket's **name** is the title in its first heading (`# <NN>: <Title>`). Refer to tickets by that name, not the bare number
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

## Ticket header

Every ticket file opens with its heading, then these plain `Key: value` lines:

```
# <NN>: <Title>

Status: ready-for-agent
Category: enhancement
Type: research
Blocked by: 01, 02
Pattern: hexagonal
```

- **`Status:`**: exactly one of
  - `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`: open states (role strings in `triage-labels.md`)
  - `claimed`: someone is working on it now
  - `resolved`: done
  - `wontfix`: will not be actioned
  - **Open** means any status other than `claimed`, `resolved`, `wontfix`. **Closed** means `resolved` or `wontfix`.
- **`Category:`**: `bug` or `enhancement`. Set by triage; optional elsewhere
- **`Type:`**: `research`, `prototype`, `brainstorming` or `task`. Wayfinder tickets only
- **`Blocked by:`**: ticket numbers (`01, 02`) or `None`. A ticket is **unblocked** when every ticket it lists is closed
- **`Pattern:`**: architecture pattern(s) the ticket follows (keys from the `architecture-patterns` table, comma-separated), or `None`. Set by `to-tickets`. Optional; when absent, whoever implements the ticket judges via `architecture-patterns`

## Operations

- **Publish / create**: create the file under `.scratch/<feature-slug>/` (creating the directory if needed)
- **Fetch**: read the file at the referenced path. The user will normally pass the path or the ticket number directly
- **Apply a role**: set the `Status:` (or `Category:`) line
- **Comment**: append under `## Comments`
- **Close**: set `Status: resolved` (done) or `Status: wontfix` (rejected or already implemented), after appending the closing comment
- **Frontier**: open tickets that are unblocked, first by number

## Wayfinding operations

Used by `/wayfinder`. The **map** is a file with one **child** file per ticket, all under `.scratch/<effort>/`.

- **Map**: `map.md` (the Destination / Notes / Decisions-so-far / Not-yet-specified / Out-of-scope body).
- **Child ticket**: `issues/NN-<slug>.md`, with the question in the body and the header above. `Type:` records the ticket type. `Status:` starts as `ready-for-agent` for an AFK ticket and `ready-for-human` for a HITL one.
- **Frontier**: scan `issues/` for open, unblocked tickets; first by number wins. `claimed` tickets are skipped.
- **Claim**: set `Status: claimed` and save before any work.
- **Resolve**: append the answer under an `## Answer` heading, set `Status: resolved`, then append a context pointer (gist + link) to the map's Decisions-so-far in `map.md`.
- **Rule out of scope**: set `Status: wontfix` on the ticket and add a line (gist + why + link) to the map's Out of scope section.
