# Glossary Format

`docs/GLOSSARY.md` is one growing, append-only file at the project root.

## Template

```md
### <term>

<1-2 sentence definition — precise enough that two people using the term mean the same thing.>
```

That's it. Add a "Not to be confused with" line only when a real, previously-seen mix-up justifies it.

## Rules

- Append new entries at the end; never rewrite, reorder, or delete existing ones.
- If a term's meaning changes, add a new entry noting the change rather than silently editing the old definition — the glossary is also a record of how the domain language evolved.
- Keep definitions free of implementation detail (no file paths, no class names) — a glossary entry describes the concept, not the code.
