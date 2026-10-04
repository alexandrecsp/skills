# Hexagonal Architecture

## Overview

Hexagonal architecture isolates business rules from the infrastructure they run on, so the rules can be tested and reasoned about without booting a database, an HTTP client, or a framework. The payoff is not "swap the adapter" — most services never actually swap providers. The real payoff is a domain layer that fast, fake-backed unit tests can cover, and an infrastructure layer that integration tests can verify independently.

## When to Use

- Structuring a new service or feature that talks to external systems (APIs, message queues, third-party SDKs) alongside its own business rules
- Reviewing code where business logic and infrastructure calls (Prisma, axios, an SDK client) live in the same class or function
- Retrofitting an existing flat/layered module into ports and adapters
- Writing tests for logic that currently requires mocking a database or HTTP client to exercise

## When Not to Use

- A thin CRUD wrapper around one table; use a lighter adapter-per-integration layout and say so explicitly
- No domain logic worth protecting: if you can't name the rules the split would make testable (Principle 1), the split buys nothing

## Core Principles

### 1. The payoff is testability, not swappability — name it before adopting the pattern

Ports-and-adapters is often reached for because "we might swap providers later." That rarely happens and isn't why the pattern earns its keep. The real win: business rules become unit-testable with in-memory fakes instead of mocked infrastructure. Before adopting the pattern, name the actual driver:

- **Testability** (business rules need to be verified without hitting Postgres/an external API) — usually justifies it.
- **Provider churn** (a second, genuinely different provider is coming, not hypothetical) — justifies it further.
- **Cargo-culting** ("this is how services are supposed to look") — doesn't justify the ceremony on its own. A thin adapter-per-integration layout without a full domain/ports split may be enough.

If you can't name 2-3 business rules the service makes that _aren't_ "call system X and reshape the response," the domain layer will be thin and the split may not be worth it yet.

### 2. One hexagon per feature, not one per service

Each bounded-context/feature gets its own `domain/ · application/ · ports/ · adapters/`, not a single service-wide set shared across unrelated features:

```
src/auth/{domain,application,ports,adapters}/
src/billing/{domain,application,ports,adapters}/
```

A service-wide hexagon invites features to share "domain" concepts that aren't actually shared, and blocks migrating or removing one feature in isolation. Cross-cutting infrastructure that legitimately serves multiple features — config, a shared DB client, a base HTTP client for a provider more than one feature calls — lives outside any single feature's hexagon, in a shared location (e.g. `src/shared/`), not duplicated per feature.

### 3. Layer ownership

```
src/<feature>/
  domain/            # no framework/ORM/SDK imports — five fixed subfolders, see below
    entities/        # classes owning their data + behavior, no suffix (token.ts)
    value-objects/   # classes, *.vo.ts
    errors/          # domain errors, *.error.ts
    services/        # pure cross-entity business logic, *.service.ts — no ports, no infra
    events/          # facts that happened, *.event.ts — plain data, no broker/queue types
  application/
    use-cases/       # atomic; the only layer that calls ports directly
    services/        # compose 2+ use-cases; nothing else lives here
  ports/             # interfaces + DI tokens — driven (outbound) side only, typed by suffix, see Principle 5
  adapters/
    http/            # controllers/handlers + DTOs — driving (inbound) side, no port needed
    <provider>/       # implements an outbound port by calling one external system
    persistence/      # implements an outbound port via the DB/ORM
  <feature>.module.ts # wiring: binds each port token to its adapter
```

`domain/` never imports the framework, an ORM client, or an SDK — that's the whole boundary. `application/` may use the framework's DI decorators (see Principle 5); it's the wiring, not the rules, that stays framework-free.

`domain/`'s five subfolders — `entities/`, `value-objects/`, `errors/`, `services/`, `events/` — are a closed, fixed set, created the moment a category has any file at all; there's no "2+ files" threshold like use-cases/services below. A single entity still lives in `domain/entities/token.ts`, not loose in `domain/`, so the layout is predictable across every feature regardless of size.

**Domain services vs. application services** — "service" means two different things depending on the layer, and the distinction matters:

- `domain/services/*.service.ts` — pure business logic that spans 2+ entities/value objects and doesn't naturally belong to one of them (e.g. a pricing rule that combines a `Cart` and a `DiscountPolicy`). Zero port or DI imports — if it needs to call a port, it isn't a domain service.
- `application/services/*` — orchestration that composes 2+ use-cases in sequence, each of which calls ports. A service that wraps exactly one use-case is a pass-through with no payoff.

```typescript
// domain/services/refund-eligibility.service.ts
export class RefundEligibilityService {
  isEligible(order: Order, policy: RefundPolicy): boolean {
    return policy.windowDays >= order.daysSincePurchase() && !order.isFinalSale();
  }
}
```

`domain/events/*.event.ts` are plain classes recording a fact that happened in the domain's own language (`OrderPlaced`, `RefundApproved`) — data only, no behavior beyond holding it, and no queue/broker types. Actually publishing one is an application-layer concern that goes through an outbound port (Principle 5) — `domain/` never imports a broker or message client.

Use-cases are atomic and touch ports directly; a use-case does one thing and is usually 1:1 with a single driving action (an HTTP route, a queue handler). Services exist only to sequence multiple use-cases together — a service that wraps exactly one use-case is a pass-through with no payoff. Let the driving adapter (a controller) call a use-case directly when there's nothing to orchestrate; reach for a service only once an action genuinely needs 2+ use-cases in sequence.

### 4. Domain entities and value objects are classes, not interfaces + free functions

Inside `domain/`, entities and value objects are classes that own their data and behaviour, and constructors enforce invariants. The rule lives in the domain-model reference: [DOMAIN-MODEL.md](../domain/DOMAIN-MODEL.md), Principles 1 and 2; apply it to everything in `domain/`. DTOs at the adapters' edge stay plain types.


### 5. Ports are driven-only, until a second driving transport actually exists

Model an interface + DI token for the **driven** (outbound) side — calls out to a database, an external API, a queue — because that's what lets a use-case test swap in a fake instead of hitting real infrastructure:

```typescript
// ports/token-repository.port.ts
export const TOKEN_REPOSITORY_PORT = Symbol('TOKEN_REPOSITORY_PORT');

export interface TokenRepositoryPort {
  getToken(): Promise<Token | null>;
  saveToken(token: Token): Promise<void>;
}
```

Don't model a symmetric port for the **driving** (inbound) side — the controller/handler that calls into the use-case — unless a second driving transport is actually in view (a queue consumer and an HTTP route both needing to reach the same use-case, say). Full symmetric hexagons are the default template in tutorials but are ceremony without a second driving caller: an HTTP controller calling a use-case class directly is already thin and cheap to integration-test as-is.

**Type every port by its suffix.** Name the file and the DI token after the *kind* of external thing the port talks to, not the specific provider — `*.repository.port.ts` for persistence, `*.gateway.port.ts` for calling another system:

```typescript
// ports/token-clock.port.ts
export const CLOCK_PORT = Symbol('CLOCK_PORT');

export interface ClockPort {
  now(): Date;
}
```

This is an open-ended naming rule, not a fixed enum — repository and gateway cover the two most common kinds, not all of them. Coin a new suffix (`*.publisher.port.ts` for pushing a domain event onto a queue, `*.clock.port.ts` for wall-clock time, `*.notifier.port.ts` for sending alerts) the moment an existing one stops actually describing what the port talks to, rather than stretching "gateway" to cover everything that isn't a database.

### 6. Pragmatic DI: the framework wires, it doesn't leak into the rules

`application/` (and `adapters/`) may use the host framework's dependency injection freely — `@Injectable()`, constructor injection, a DI container binding a port token to its adapter class. Making the domain "framework-agnostic" (manual wiring, no decorators, in case it's ever lifted out of the framework) is rarely worth the ceremony when there's no actual second runtime in view. The boundary that matters is `domain/` staying free of infrastructure imports — not the application layer staying free of the framework's own DI.

```typescript
// application/use-cases/get-valid-token.use-case.ts
@Injectable()
export class GetValidTokenUseCase {
  constructor(
    @Inject(TOKEN_REPOSITORY_PORT) private readonly tokens: TokenRepositoryPort,
    @Inject(OAUTH_PORT) private readonly oauth: OAuthPort,
  ) {}

  async execute(): Promise<Token> {
    const current = await this.tokens.getToken();
    if (current && !current.isExpired()) return current; // Token.isExpired(): domain method, see DOMAIN-MODEL.md
    const refreshed = await this.oauth.refresh(current.refreshToken);
    await this.tokens.saveToken(refreshed);
    return refreshed;
  }
}
```

### 7. Domain errors are transport-agnostic; adapters translate them at the edge

Domain and application errors are business failures and carry no transport detail; the driving adapter translates them once, at its edge (an exception filter, an HTTP middleware, a queue handler's catch block). The rule lives in the domain-model reference: [DOMAIN-MODEL.md](../domain/DOMAIN-MODEL.md), Principle 3. This is what lets a use-case test assert on the domain error type directly.


### 8. Fakes for use-cases, integration tests for adapters

Two different layers get two different test strategies — don't blur them:

- **`application/use-cases/*`**: unit tests inject **fakes** of each port (a small in-memory class implementing the interface), not mocks/spies. A fake exercises the actual port contract; a mock only records that a method was called, and silently tolerates the fake and the real adapter drifting apart.
- **`adapters/*`**: tested as **integration tests** against the real dependency (a dockerized DB, a sandboxed external API, or recorded fixtures) — this is the only layer actually verifying the adapter does what its port promises. Mocking axios/the ORM client inside an adapter test verifies nothing but that the mock was configured correctly.

```typescript
// application/use-cases/get-valid-token.use-case.spec.ts
class FakeTokenRepository implements TokenRepositoryPort {
  private token: Token | null = null;
  async getToken() { return this.token; }
  async saveToken(t: Token) { this.token = t; }
}
```

### 9. Promote to shared infrastructure the moment a second consumer is known — don't wait for pressure

When a piece of infrastructure (a base HTTP client for a provider, a DB connection module) is known to serve a second feature soon, move it to the shared location now rather than leaving it feature-owned until the second feature forces an extraction under pressure. Waiting means touching every existing import at the least convenient time; moving early costs nothing because nothing depends on the old location yet.

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "We might swap providers later" | Rarely happens, and isn't the real payoff — see Principle 1. Testability is. If you can't name the domain logic being protected, you may not need the full split yet. |
| "Let's model ports for both directions, hexagonal tutorials do" | Driving-side ports are ceremony without a second driving transport (Principle 5). A controller calling a use-case class directly is already cheap to test. |
| "The domain should be framework-agnostic in case we migrate frameworks" | Framework migration is rarely a live threat; paying for it in every DI wiring is Principle 6's cargo-cult trap. Keep `domain/` free of infra imports — that's the boundary that matters. |
| "One shared domain/ for the whole service keeps things consistent" | Invites unrelated features to share concepts that aren't actually shared, and blocks isolating one feature later (Principle 2). |
| "We'll extract the shared client when the second feature needs it" | Extracting under pressure means touching every existing import at the worst time. Promote it now if the second consumer is already known (Principle 9). |
| "Mocking Prisma/axios in the use-case test is fine, it's just for coverage" | A mock only proves the mock was called; a fake proves the port's actual contract. Mocking infra in a use-case test also usually means the port boundary isn't actually being used (Principle 8). |
| "This service is small, hexagonal is overkill" | Sometimes true — see Principle 1. Say so explicitly and use a lighter adapter-per-integration layout instead of adopting the pattern out of habit either way. |
| "domain/ is small, doesn't need entities/value-objects/errors/services/events split out" | The five subfolders are fixed and cheap — even one file gets its own subfolder (`domain/entities/token.ts`), so the layout stays predictable across every feature regardless of size (Principle 3). |
| "This logic doesn't call any ports, it can live in application/services/ next to the orchestration code" | If it doesn't touch a port, it isn't orchestration — it belongs in `domain/services/` as pure logic, testable without even a fake (Principle 3). |
| "Repository and gateway cover every port we have" | They cover the two most common kinds, not all of them — coin a new suffix (publisher, clock, notifier...) that names what the port actually talks to, rather than stretching gateway to cover it (Principle 5). |

## Verification

After structuring or reviewing a hexagonal boundary:

- [ ] The actual driver (testability vs. provider churn vs. none) is named, not assumed
- [ ] `domain/` and use-cases have zero imports of the framework's infra APIs, an ORM client, or an SDK
- [ ] The domain-model checklist in [DOMAIN-MODEL.md](../domain/DOMAIN-MODEL.md) passes for everything in `domain/`
- [ ] Ports exist for the driven (outbound) side; driving (inbound) side only has a port if a second driving transport is real
- [ ] Every port has a second implementation or a fake that a test uses; a port that exists "because hexagonal" has neither
- [ ] Every use-case is atomic and 1:1-able with a driving action; every service composes 2+ use-cases (never fewer)
- [ ] Use-case unit tests inject fakes of each port; adapter tests are integration tests against the real dependency or fixtures
- [ ] Each feature owns its own `domain/application/ports/adapters`; cross-feature infrastructure lives in a shared location, not duplicated
- [ ] Infrastructure with a known second consumer has already been promoted to the shared location
- [ ] `domain/` files are split into `entities/value-objects/errors/services/events`, each in its dedicated subfolder
- [ ] `domain/services/*` has zero port/DI imports; `application/services/*` only composes 2+ use-cases
- [ ] Every port file's suffix names the kind of external system it talks to (repository, gateway, or a freshly coined kind), never left generic
