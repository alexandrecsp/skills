# UI prototype

Several **structurally different** variants of one UI on a single route, switched from a floating bottom bar. The user flips between them in the browser, picks one or combines parts of several, and the rest are thrown away.

## Host page

A variant is judged best **in situ**: against the app's real header, sidebar, data and density. On an empty route every variant looks fine.

- **Existing page (the default).** Render the variants on the route that already exists, gated by a `?variant=` search param. Data fetching, params and auth stay as they are; only the rendered subtree swaps. Something that would naturally live inside an existing page (a new dashboard section, a settings card, a step in a flow) mounts inside that page too.
- **New page (only when nothing can host it).** For an entirely new top-level surface, create a throwaway route under the project's routing convention, with `prototype` in its path, using the same `?variant=` param.

## Process

1. **State the question and the plan.** Default to 3 variants, at most 5: past that they stop being structurally different. Give each variant a **structural constraint** the others don't share: a layout, information hierarchy or primary affordance ("sidebar-led", "one scrolling column", "table first, no cards"). Write the plan in one line as a top-of-file comment, such as "Three variants of the settings page (sidebar-led, one column, table first), switched by `?variant=`, on the existing `/settings` route." Done when the plan names the host page and each variant's constraint.

2. **Draft the variants.** Spawn one `alper-skills:designer` agent per variant, in parallel. A separate context per variant keeps each one from drifting toward the others' layouts. Each brief carries:

   - the question, and the host page: its route, its data and props, and the files around it
   - the project's component library and styling, by file path
   - the variant's file path, and its exported name (`VariantA`, `VariantB`)
   - the variant's structural constraint from step 1
   - the variant rules below, and the rules for both branches from [SKILL.md](SKILL.md), pasted in

   The variant rules: variants differ in **structure**: layout, information hierarchy, primary affordance. Colour and copy alone make a tweak, not a variant. Each variant owns its layout, sharing at most small pieces such as a header. Variants stay read-only; one that needs a mutation calls a stub.

   Each designer returns its variant's short name for the floating bar (`Sidebar layout`) and one line on its structure. Done when every designer has returned and no two variants share a layout; redispatch one with a sharper constraint ("no card grid") when two come out alike.

3. **Wire the switcher.** One switcher on the route reads the param and renders the variant:

   ```tsx
   // adapt to the project's framework
   const variant = searchParams.get('variant') ?? 'A';
   return (
     <>
       {variant === 'A' && <VariantA {...data} />}
       {variant === 'B' && <VariantB {...data} />}
       {variant === 'C' && <VariantC {...data} />}
       <PrototypeSwitcher variants={['A', 'B', 'C']} current={variant} />
     </>
   );
   ```

   Done when each `?variant=` key renders its variant.

4. **Build the floating bar.** One shared component, placed where the project keeps shared UI: a fixed pill at bottom centre with a left arrow, the current key plus the variant's name (`B (Sidebar layout)`), and a right arrow, both wrapping around.

   - Arrows update the search param through the framework's router (`router.replace`, `navigate`), so a variant is shareable and survives a reload.
   - The `←` and `→` keys cycle too, except while an `<input>`, `<textarea>` or `[contenteditable]` has focus.
   - It looks clearly apart from the design under review: high contrast, a soft shadow.
   - It renders only outside production (`process.env.NODE_ENV !== 'production'` or the equivalent), so a stray merge cannot ship it.

   Done when the arrows and keys cycle every variant and the URL follows.

5. **Hand it over.** Give the user the URL and the `?variant=` keys. The best feedback is usually "the header from B with the sidebar from C": that combination is the design they want. Done when a variant, or a combination, has won.

6. **Capture.** Follow the capture in [SKILL.md](SKILL.md): every variant and the switcher go to the prototype branch, and the verdict names the winner and why. The winner is rebuilt properly in the real page; its prototype code was written without tests or error handling.
