# Backend

A server with no UI of its own: an API, a worker, a CLI. Each module is a full hexagon; driving adapters are the transports that call in. The tree is TypeScript; Java and Python spell it as [SKILL.md](SKILL.md)'s Names say.

```
src/
├── main.ts                      ← composition root: wires modules, starts the transports
├── shared/
└── modules/ordering/
    ├── index.ts                 ← the module's interface: use cases, application DTOs and published events
    ├── domain/
    │   ├── entities/order.ts
    │   ├── value-objects/money.ts
    │   ├── ports/order-repository.port.ts
    │   └── events/order-placed.event.ts
    ├── application/
    │   ├── use-cases/
    │   │   ├── place-order.use-case.ts
    │   │   └── place-order.use-case.test.ts
    │   ├── services/pricing.service.ts
    │   └── dtos/
    │       ├── place-order.command.ts
    │       └── order.result.ts
    └── infrastructure/
        ├── driving/
        │   ├── order.controller.ts
        │   ├── order.listener.ts         ← queue handler
        │   └── dtos/
        │       ├── place-order.request.ts
        │       └── order.mapper.ts
        ├── driven/
        │   ├── postgres-order-repository.adapter.ts
        │   ├── in-memory-order-repository.adapter.ts
        │   └── dtos/postgres-order.model.ts
        └── config/ordering.config.ts
```

- **A driving adapter is thin**: validate its request, map it to a command, call one use case, map the result or domain error to its response. Business rules found there belong in the domain.
- **Schemas and migrations** belong to the driven adapter that owns the store: a technology with several files gets its own folder, `infrastructure/driven/postgres/`, holding its adapters, models and migrations.
- **Framework modules** (NestJS `*.module.ts`) are the module's wiring and live in `infrastructure/config/`.
