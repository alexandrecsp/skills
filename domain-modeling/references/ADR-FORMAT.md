# ADR Format

ADRs live in `docs/adr/` and use sequential numbering: `0001-slug.md`, `0002-slug.md`, etc.

Create the `docs/adr/` directory lazily: only when the first ADR is needed.

## Template

```md
# {Short title of the decision}

{1-3 sentences: what's the context, what did we decide, and why.}
```

That's it. An ADR can be a single paragraph. The value is in recording *that* a decision was made and *why*, not in filling out sections.

## Optional sections

Only include these when they add genuine value. Most ADRs won't need them.

- **Status** frontmatter (`proposed | accepted | deprecated | superseded by ADR-NNNN`): useful when decisions are revisited.
- **Considered Options**: only when the rejected alternatives are worth remembering.
- **Consequences**: only when non-obvious downstream effects need to be called out.

## Numbering

Scan `docs/adr/` for the highest existing number and increment by one.

## Superseding

Never edit a past ADR's decision text. If a decision is reversed, write a new ADR and mark the old one's Status as `superseded by ADR-NNNN` (add the Status section retroactively if it didn't have one).
