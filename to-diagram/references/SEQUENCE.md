# Sequence diagrams

A sequence diagram answers one question: *who calls whom, in what order, and what comes back*. Time and order are the information it exists to carry — if the important thing is a decision tree inside one component, or the static shape of a system, sequence is the wrong tool (see the router's type table). Reach for it when a request crosses 3+ real components, when concurrency/timeouts/races matter, or when you're documenting one concrete use case's behavior, not a whole feature's structure.

## Scope: one scenario per diagram

A sequence diagram covers one concrete scenario, not a feature. "User logs in with valid credentials" is a scope; "the authentication system" is not — that includes login, logout, refresh, password reset, and MFA, and turns into soup. Test: you should be able to describe the diagram in one sentence — "shows what happens when [actor] does [specific action] and [specific result]." If you can't, the scope is too wide; split it.

Signs a diagram is trying to do too much: more than ~15-20 messages total, needing both horizontal and vertical scrolling to see it, two or more `alt` branches covering unrelated business scenarios (success vs. validation error vs. timeout vs. third-party failure), or catching yourself wanting to add a legend explaining which section is which. If you'd tell a reader "ignore this part, focus on that one," it should already be two diagrams.

## Participants

**Name by role, at one consistent abstraction level.** Not generic (`System`, `Backend` — says nothing) and not implementation-specific (`OrderServiceImpl.processPaymentInternalV2()` — leaks detail that breaks on the next refactor and means nothing to the reader). Name participants the way you'd name them in the architecture style table in the router's `SKILL.md` — `Order Service`, `Payment Service`, `Postgres (orders)` for an engineering audience; `Customer`, `Online Store`, `Card Processor` for a business audience. Never mix levels in one diagram (a `Customer` next to a `PaymentServiceGrpcClient` is the single most-cited inconsistency in review). Test: the name should survive a refactor — if renaming an internal class breaks the diagram's legend, the name was too specific.

**Order left to right by causal flow**: whoever triggers the scenario goes leftmost; as the call propagates inward, participants go right, usually ending at the most passive resource (database, queue, external system). This is what keeps arrows from crossing — arbitrary or alphabetical ordering is the number one cause of a tangled diagram.

**`actor` is for humans only** — the stick figure is reserved for a person or an external role making a decision (end user, operator). Every piece of software, including a third-party API, is a `participant` (box) — it's a system being called, not a person deciding something.

**Keep it to 5-7 participants.** Past that, the eye loses track of the horizontal read even when the logic itself isn't complex. Going over usually means the scope is too wide (see above), or some participants are pure pass-through (a proxy, a load balancer) that don't need their own lifeline unless their behavior — retry, circuit-breaking — is the actual point of the diagram.

## Messages

- **Sync** (`->>`) means the caller blocks and waits. **Async** (a fire-and-forget message, not a returning call) means the caller moves on immediately. This distinction is not stylistic — it asserts something real about coupling, timeout behavior, and resilience. Drawing a queue publish as if it were synchronous wrongly implies the publisher waits for the consumer to finish.
- **Return** (`-->>`): draw it explicitly when the returned value carries information the reader needs (a value used in a later decision, an error that triggers an `alt`, data referenced in a note). Skip it for a trivial call whose result nothing downstream depends on — drawing every return doubles the diagram's lines for no information gain.
- **Self-messages** (`A->>A: validate rules`) are fine when that internal step matters to the reader (a named business rule, a decision affecting what happens next). Never draw one for generic "process data" — either name the specific rule or cut it.
- **Granularity — the vanishing-line test**: for every candidate message, ask "if this line disappeared, would the reader lose understanding of the system's behavior?" If not, cut it. This is how you decide the right zoom level for the whole diagram, and keep it consistent start to finish: don't mix a high-level call (`Order Service ->> Payment Service: charge customer`) with a low-level one (`Payment Service ->> Payment Service: paymentRepo.findById(id)`) in the same diagram — that's an abstraction-level jump, one of the most common defects in review. Skip getters/setters, trivial format validation, and logging/metrics calls (unless the diagram is specifically about observability). Show business decisions, every network/system boundary crossing (that's exactly what this diagram type is for — those calls have latency and can fail), and real side effects (a write, a published event, a sent email).

## Activation bars

An activation bar shows who's "holding the ball" and for how long, logically — not a real-time duration. In Mermaid, activation is implicit by default (a `->>` opens one, closing on the matching `-->>`), or explicit with `+`/`-` suffixes:

```
Client->>+API: POST /orders
API->>+DB: INSERT order
DB-->>-API: OK
API-->>-Client: 201 Created
```

Use explicit activation when you need a bar to span an entire `loop` or `par` block rather than one call, or to show two overlapping activations on the same participant (concurrent/reentrant calls, stacked side by side).

Common mistakes: leaving a bar without its matching close (confusing in tools with manual activation control); activating a purely passive participant (a database that just answers one query) for the whole diagram's span, which visually overstates how much "work" it's doing; and sizing or annotating a bar to suggest real elapsed time — a sequence diagram isn't a timing diagram. If real duration matters, say so in a note with an actual number, don't imply it visually.

## Combined fragments

- **`alt / else`**: 2+ mutually exclusive paths the reader needs to compare side by side. Fine inline when each branch is short (~4-5 messages or fewer).
- **`opt`**: a short conditional detour with no real "else" — "this also happens sometimes" (an optional push notification), not error handling.
- **`loop`**: always label the stop condition (`loop up to 3 attempts`, `loop for each cart item`) — an unlabeled loop is ambiguous about whether it's bounded.
- **`par ... and ... end`**: only when the concurrency itself is the point (two independent effects genuinely happening at once, neither depending on the other's result). Don't reach for it just to visually group unrelated sequential messages.
- **`critical ... option ... end`**: rare outside audiences already familiar with the notation (atomic sections with fallback) — a plain note often communicates the same intent more clearly to a general reader.
- **`break`**: underused. When a condition aborts the rest of the normal flow entirely, `break` says "stop here" more directly than an `alt` with an empty branch.

**Nesting limit: two levels.** An `alt` inside a `loop` is normally still readable; a third level (`alt` inside `opt` inside `loop`) is where automatic rendering turns into boxes-within-boxes that push everything right and tall enough to lose the reader. If the logic genuinely needs 3+ levels, extract it: Mermaid has no native cross-diagram reference (unlike PlantUML's `ref`), so point at it with a note — `Note over Client,API: see "Payment Retry" diagram` — instead of forcing the nesting inline.

**Deciding if an error path deserves an `alt` or its own diagram**: ask whether the error is part of the same story or has its own cause/effect/recovery arc. A short, symmetric error (`invalid credentials → 401`) fits a 1-2 line `alt` right next to the happy path. An error that triggers its own retries, compensation, or notifications to other systems is a candidate for a separate diagram. Numeric tell: if an `alt` branch has more messages than the happy path around it, promote it to its own diagram.

## Notes

A note earns its place when it carries something the arrows can't — the *why*, not the *what*: a domain invariant ("reservation locks the item for 15 minutes before auto-expiring"), a non-obvious business rule ("refunds over $500 require manual approval before this step"), or a relevant architectural fact ("idempotency guaranteed via the `Idempotency-Key` header"). A note that just restates what the arrow's label already says is noise — cut it. Keep notes to 1-2 lines; a paragraph-sized note belongs in the accompanying text, not squeezed into the diagram.

## Splitting into multiple diagrams

Split when: the flow has a genuinely distinct business branch (see Scope above), or a setup/auth handshake is reused across several flows (extract it once, reference it by note from the others instead of repeating the same 5 messages every time). Rule of thumb: if presenting it requires saying "skip this part for now," it should already be two diagrams.

## Output format

- An H2 title naming the specific scenario (not the feature) — e.g. "User Login — Valid Credentials", not "Authentication".
- The ```mermaid code block.
- A short list below it naming each participant and its real file/class/system reference.

## Example

Scenario: "user redeems an item from inventory, already redeemed."

```mermaid
sequenceDiagram
    actor User
    participant Ctrl as Inventory Controller
    participant UC as Redeem Item (UseCase)
    participant Dom as Inventory (Domain)
    participant Repo as Inventory Repository

    User->>+Ctrl: Redeem item (itemId)
    Ctrl->>+UC: execute(RedeemItemCommand)
    UC->>+Dom: redeem(itemId)
    Note right of Dom: Invariant: item must be<br/>owned and unredeemed
    alt item is redeemable
        Dom-->>UC: RedemptionResult(success)
        UC->>+Repo: save(inventory)
        Repo-->>-UC: OK
        UC-->>-Ctrl: success
        Ctrl-->>-User: 200 OK
    else already redeemed
        Dom-->>-UC: RedemptionResult(failure)
        UC-->>-Ctrl: error
        Ctrl-->>-User: 409 Conflict
    end
```

Participants:
- `Inventory Controller` — `InventoryController.java`, inbound HTTP adapter
- `Redeem Item` (UseCase) — `RedeemItemUseCase.java`
- `Inventory` (Domain) — `Inventory.java`, owns the redemption invariant
- `Inventory Repository` — `InventoryRepository.java`, persistence

## Common rationalizations

| Rationalization | Reality |
|---|---|
| "I'll cover the whole feature in one diagram, it's more complete" | A diagram covering 2+ unrelated business scenarios is soup — split by scenario, not by feature. |
| "I'll draw every return arrow, it's more accurate" | Accurate but unreadable — draw returns only when the value matters downstream. |
| "This error path fits as an `alt`, no need for a new diagram" | Fine if it's short and symmetric to the happy path; if it has its own retries/compensation and outnumbers the happy path, it's its own diagram. |
| "Getters and internal validation calls show I did my homework" | They show noise — the vanishing-line test cuts anything the reader wouldn't miss. |

## Red flags

- Mixed abstraction levels in one diagram (a business-level call next to a repository method call)
- A diagram trying to tell 2+ unrelated stories (happy path + three unrelated error scenarios + retry + fallback, all nested)
- A "god" participant appearing in every single message (sometimes a modeling mistake, sometimes an honest reflection of real excessive coupling worth flagging)
- Unlabeled or genericly-labeled arrows (`process()`, `handle()`)
- Every return arrow drawn regardless of whether the value matters
- Sync/async notation used incorrectly, misrepresenting real blocking behavior
- Arbitrary participant order causing crossed lines
- A `loop` with no visible stop condition
- Fragment nesting past two levels
- A diagram documenting static structure (who depends on whom) with no real temporal scenario — that's a C4/component diagram wearing a sequence diagram's syntax

## Verification

- [ ] The diagram covers one concrete scenario, describable in one sentence
- [ ] Every participant name is real, at one consistent abstraction level throughout
- [ ] Participants are ordered to minimize crossed lines; no more than ~7
- [ ] Every message passes the vanishing-line test; sync/async/return notation matches real behavior
- [ ] Fragments nest no more than two levels deep; every `loop` has a visible stop condition
- [ ] Notes carry business/domain context the arrows can't — none just restate a label
- [ ] Output includes the H2 title (scenario-specific), the ```mermaid block, and the participant list
