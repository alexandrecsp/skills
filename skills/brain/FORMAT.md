# Brain format

The shape of the second brain, shared by the `brain` skill's branches and the graph page, [graph.html](graph.html), which parses it. The notes stay Obsidian-compatible: opened as a vault, the brain's folder gets Obsidian's graph view too.

## Layout

```
<brain>/
├── BRAIN.md                    ← what the brain is, how to join, the repositories it covers
├── repos/
│   └── <repo-slug>/<name>.md   ← notes on one repository
└── shared/<name>.md            ← notes spanning several repositories
```

A **repo slug** names a repository the same way on every teammate's machine, whatever folder each clone sits in. It is the slug named in the repository's **Second brain** section of `AGENTS.md` or `CLAUDE.md`, when there is one; else the last segment of `git remote get-url origin` without `.git`; else the root folder's name. It is kebab-case.

Each machine records where its clone of the brain lives in `~/.alper/brain.json` (`~` is `%USERPROFILE%` on Windows): `{ "path": "<absolute path, forward slashes>" }`. A repository is **registered** when that file names a brain holding `repos/<slug>/` for it; `/brain setup` registers it. Every skill outside `brain` leaves the brain alone in a repository that is not registered.

## Notes

One fact per file. Teammates adding notes at the same time never conflict, and the graph shows one node per fact.

```markdown
---
name: billing-retry-idempotency
description: Billing retries reuse the charge's idempotency key, so a retried charge never bills twice
type: gotcha
---

A retried charge sends the first attempt's idempotency key, and the provider answers with the first attempt's result.

**Why:** the provider double-billed during the March outage, when retries carried fresh keys.
**How to apply:** pass the stored key to every retry path, including manual replays from the admin console.

Related: [[invoice-event-contract]]
```

- `name`: the file name without `.md`, in kebab-case, unique across the **whole brain** and not only its folder, since `[[name]]` resolves by it here and in Obsidian. Prefix the repo slug when a bare name could clash.
- `description`: one line that stands alone. Recall judges relevance from it without opening the note.
- `type`: one of the types below.
- `repos`: in `shared/` only, the slugs the note spans, as `[billing-api, web-app]`. A note in `repos/<slug>/` belongs to that slug.

| type | holds |
|---|---|
| `concept` | how a part works, or what a term means in this repository |
| `decision` | a choice and its reason. When the repository keeps ADRs, link the ADR's path and record only what it lacks |
| `gotcha` | a trap: surprising behaviour, or a past incident and its lesson |
| `contract` | an interface between repositories (an API, event, queue or shared schema) with its producer and consumers. Lives in `shared/` |
| `how-to` | a recurring procedure that no script or doc in the repository runs |

The body states the fact first. A `decision` or `gotcha` follows it with **Why:** and **How to apply:** lines. Link every note it builds on or bears on with `[[name]]`: each link is an edge in the graph. A link to a note not written yet draws as a dashed node, marking a note worth writing.
