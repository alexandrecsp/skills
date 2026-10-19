# Logic prototype

One self-contained HTML file, a **shareable demo**, in which anyone drives a state model by clicking buttons. It answers questions about business logic, state transitions or data shape: the kind that look reasonable on paper and only feel wrong once pushed through real cases. Because it opens by double-click with nothing to install, a designer, PM or domain expert can feel the model for themselves, so it speaks their language, not the code's.

## Process

1. **State the question.** Write the state model and the question in one paragraph, shown as a visible intro at the top of the page. It lets the user check, now or later, that the prototype answers the right question. Done when the paragraph names the model and the question.

2. **Write the pure module.** Put the logic in one `<script>` block as a small **pure** module that could be lifted into the real codebase: the page calls into it, and it never touches the DOM, `document` or a handler. Pick the shape the question needs, not the one easiest to wire:

   - **A reducer**, `(state, action) => state`, when actions are discrete events over one state value.
   - **A state machine**, with explicit states and transitions, when which actions are legal right now is part of the question.
   - **A set of pure functions** over a plain data type, when there is no current state, only transformations.
   - **A class with a clear method surface**, when the logic genuinely owns ongoing internal state.

   Done when every action the question needs exists in the module, and the module holds no reference to the page.

3. **Build the page.** Plain HTML, CSS and JS, all inline: no framework, bundler or server, so it survives being emailed. Every label is in **domain language**, the business's words rather than the reducer's. Top to bottom:

   1. **Title and one-line explanation** of what the demo explores (the question from step 1).
   2. **Current state**: the full relevant state as a readable panel of labelled fields, re-rendered after every click, with what just changed called out.
   3. **Free-play buttons**: one per action, always available, so anyone can poke the model in any order.
   4. **Guided walkthroughs**: one **scenario** per tab, each a short plain-language description (the situation it sets up, what to watch for) above its ordered buttons. Each step is a real button that performs its action and moves to the next step. Starting a walkthrough resets to a known initial state, so it runs the same way every time.

   Choose scenarios for the awkward cases: the happy path, a tricky edge case, an attempt at something that should be illegal. Style it clean and restrained: clear typography, generous spacing, one accent colour, no animation. Done when every scenario runs start to finish from its tab.

4. **Hand it over.** Send the file's path, or open it. The valuable moments are "wait, that shouldn't be possible" and "huh, I assumed X": those are bugs in the _idea_, the whole point. Add actions or scenarios as the user asks. Done when the question has a verdict.

5. **Capture.** Follow the capture in [SKILL.md](SKILL.md). The pure module is the part of the prototype worth reading again when the real module is built; the page around it stays on the prototype branch.
