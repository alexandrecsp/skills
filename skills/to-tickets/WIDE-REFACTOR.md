A **wide refactor** is one mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase: a single edit breaks every call site at once, so no tracer bullet can land green. Sequence it as **expand–contract**, all inside one milestone:

1. **Expand**: add the new form beside the old, so nothing breaks.
2. **Migrate**: move the call sites over in batches sized by blast radius (per package, per directory), one ticket per batch, each blocked by the expand. The old form still exists, so every batch lands green.
3. **Contract**: delete the old form once no caller remains, in a ticket blocked by every migrate batch.

When even the batches cannot land green alone, keep the sequence but let the batches share an integration branch, all blocking a final integrate-and-verify ticket; green is promised only there.

The milestone is done when the contract ticket lands.
