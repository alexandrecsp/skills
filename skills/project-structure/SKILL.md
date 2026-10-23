---
name: project-structure
description: Hexagonal folder layouts and file names for frontend, backend and full-stack apps, in any language. Use when creating a project, placing or naming a new file, or recording a repo's layout.
metadata:
  type: reference
---

Lay code out as a **hexagon** per module (Ports and Adapters, Cockburn): the domain at the centre, the application around it, infrastructure at the edge. One layout serves every language; only the casing of names changes. This skill is the default; a repo's `docs/STRUCTURE.md` records its own layout and always wins over it.

## Vocabulary

Use these terms exactly; `port` and `adapter` mean what they mean in `codebase-design`.

- **Module**: a feature-sized slice named by a `GLOSSARY.md` term, holding its own hexagon. Code is grouped by module first, layer second.
- **Domain**: the module's rules: entities, value objects, and the driven ports they need, plus domain events and errors. Pure: no framework, no I/O.
- **Aggregate**: a root entity and the entities and value objects it guards, loaded and saved as one. Each root gets one repository port.
- **Driven port**: an interface the module needs from the outside (a repository, a gateway, a clock, a unit of work). It lives in the domain, so every port has one home.
- **Application**: the module's use cases, the services they share, and the DTOs they take and return.
- **Use case**: the only entry point for one user intent, and the module's **driving port**: adapters call it directly, through no separate interface.
- **Service**: application logic shared by two or more use cases, called only by use cases. A service with one caller folds back into it.
- **DTO**: data shaped to cross a boundary, owned by the side that defines the boundary.
- **Infrastructure**: the adapters, plus the module's wiring. A **driving adapter** turns an outside call (HTTP, CLI, UI event, queue message, framework route) into a use-case call; a **driven adapter** implements a driven port with a technology (Postgres, Stripe, `fetch`, in-memory).
- **Composition root**: the one place per app that wires driven adapters into use cases.
- **Shared kernel**: `shared/`, code every module may use; it imports from no module.

## Layout

```
<module>/
├── domain/
│   ├── entities/
│   ├── value-objects/
│   ├── ports/            ← driven ports
│   ├── events/
│   └── errors/
├── application/
│   ├── use-cases/        ← driving ports
│   ├── services/
│   └── dtos/             ← commands and results
└── infrastructure/
    ├── driving/
    │   └── dtos/         ← requests and responses, with their mappers
    ├── driven/
    │   └── dtos/         ← persistence models and external payloads, with their mappers
    └── config/           ← the module's wiring
```

- **Dependencies point inward**: infrastructure → application → domain. The domain imports only the domain and the shared kernel.
- **Use cases take commands and return results.** Driving adapters map their DTOs to and from these, so they never hold a domain entity. Driven adapters implement domain ports in entity terms and map each entity to and from their own model.
- **The domain holds no DTOs**: entities and value objects carry behaviour, and DTOs live at the boundaries around them.
- **A module's interface is its use cases, application DTOs, and the domain events it publishes.** Another module imports only those, through the module's public entry (`index.ts`, `__init__.py`, the packages [JAVA.md](JAVA.md) names).
- **Every driven port has two adapters**: production, and the in-memory one use-case tests run on (`codebase-design`'s "two adapters mean a real one").
- **A folder exists once it holds a file.** A display-only frontend module can be just `infrastructure/driving/`.
- **Tests sit beside the file they test**, where the language allows; use-case tests run at the application seam with in-memory driven adapters.

## Names

Entities and value objects take the plain glossary noun: they are the domain's language. Every other file names its role with a suffix, so its layer reads from a tab title or a search hit, and every role is one glob away.

| Folder | Role suffix | Example |
|---|---|---|
| `domain/entities` | none | Order |
| `domain/value-objects` | none | Money |
| `domain/ports` | Port | OrderRepository Port |
| `domain/events` | Event | OrderPlaced Event |
| `domain/errors` | Error | InsufficientStock Error |
| `application/use-cases` | UseCase | PlaceOrder UseCase |
| `application/services` | Service | Pricing Service |
| `application/dtos` | Command, Query, Result | PlaceOrder Command, Order Result |
| `infrastructure/driving` | the channel: Controller, Listener, Cli, Actions | Order Controller |
| `infrastructure/driving/dtos` | Request, Response | PlaceOrder Request |
| `infrastructure/driven` | Adapter, prefixed by technology | Postgres OrderRepository Adapter |
| `infrastructure/driven/dtos` | Model (persistence), Payload (external API), prefixed by technology | Postgres Order Model |
| any `dtos` | Mapper | Order Mapper |

Each language spells the same name its own way:

- **TypeScript**: kebab-case, the suffix after a dot: `order.ts`, `order-repository.port.ts`, `place-order.use-case.ts`, `postgres-order-repository.adapter.ts`, `place-order.request.ts`. Tests: `<file>.test.ts`. Folders kebab-case.
- **Python**: snake_case, the suffix after an underscore: `order.py`, `order_repository_port.py`, `place_order_use_case.py`. Folders snake_case (`value_objects/`); tests in the module's `tests/`, mirroring its folders, named `test_<file>.py`.
- **Java**: PascalCase class names, packages for folders: [JAVA.md](JAVA.md).

A name the framework reserves (`page.tsx`, `route.ts`, `+page.svelte`) keeps the framework's name: the folder makes it a driving adapter.

## Archetypes

Detect the archetype from the manifests, then read its file:

- **Full-stack**: a framework that serves UI and server code from one app (`next`, `nuxt`, `@remix-run/*`, `@sveltejs/kit`). [FULLSTACK.md](FULLSTACK.md).
- **Frontend**: a UI framework with no server of its own (`react`, `vue`, `svelte` with `vite`). [FRONTEND.md](FRONTEND.md).
- **Backend**: a server framework or none (`express`, `fastify`, `@nestjs/core`, `spring-boot`, `django`, `fastapi`). [BACKEND.md](BACKEND.md).

A monorepo gets an archetype per app or package (`apps/web` frontend, `apps/api` backend), each laid out on its own.

## Recording

`docs/STRUCTURE.md` is the repo's source of truth for layout, committed with the code. Write it as: the dependency rules and departures every app shares, then one section per app headed by its path (`apps/web`), each with its archetype and language, its tree, the names table in its language's spelling, and its own departures, every departure with its reason. Read the section of the app you're placing a file in. In a repo with code, record the layout it already uses, mapped onto this vocabulary where it fits; moving existing code is a refactor for `improve-and-refactor`, never part of recording.
