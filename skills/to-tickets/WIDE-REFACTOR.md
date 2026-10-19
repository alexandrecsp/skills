A **wide refactor** is one mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase: a single edit breaks every call site at once, so no tracer bullet can land green. Sequence it as **expand–contract**, all inside one milestone:

1. **Expand**: add the new form beside the old, so nothing breaks.
2. **Migrate**: move the call sites over in batches sized by blast radius (per package, per directory), one ticket per batch, numbered after the expand. The old form still exists, so every batch lands green.
3. **Contract**: delete the old form once no caller remains, in a ticket numbered after every migrate batch.

When a batch cannot land green alone, merge it with the batches it interlocks with until each lands green. When a merged batch no longer fits one fresh context window, flag it to the user in the quiz.

The milestone is done when the contract ticket lands.
