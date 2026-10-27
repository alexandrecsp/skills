Explore alternative interfaces for a module with parallel sub-agents. Based on "Design It Twice" (Ousterhout): your first idea is unlikely to be the best.

## Process

1. **Frame the problem space.** Write the user a short explanation of the problem: the constraints any new interface must satisfy, the dependencies it relies on with their categories from [DEEPENING.md](DEEPENING.md), and a rough code sketch that makes the constraints concrete (an illustration, not a proposal). Show it, then go straight to step 2: the user reads while the sub-agents work. Done when the user has the explanation.

2. **Spawn sub-agents.** Spawn 3 or more `alper-skills:designer` agents in parallel, each with its own technical brief (file paths, coupling details, dependency categories, what sits behind the seam) and a different design constraint:

   - Agent 1: "Minimise the interface: 1 to 3 entry points, maximum leverage per entry point."
   - Agent 2: "Maximise flexibility: support many use cases and extension."
   - Agent 3: "Optimise for the most common caller: make the default case trivial."
   - Agent 4, when a dependency crosses the network: "Design around Ports and Adapters for cross-seam dependencies."

   Paste the vocabulary from [SKILL.md](SKILL.md), plus the glossaries of the [domain docs](../domain-modeling/DOMAIN-DOCS.md) when present, into every brief, so each design names things in the same terms. Each sub-agent returns:

   1. The interface (types, methods, parameters, plus invariants, ordering, error modes)
   2. A usage example from a caller
   3. What the implementation hides behind the seam
   4. Its dependency strategy and adapters
   5. Trade-offs: where leverage is high, where it's thin

   Done when every sub-agent has returned a design.

3. **Compare and recommend.** Present the designs one at a time, then compare them in prose by **depth**, **locality**, and **seam placement**. Recommend the strongest, or a hybrid when parts of several combine well: the user wants a strong read, not a menu. Done when the user has your recommendation.
