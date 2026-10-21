# Full-stack

One app serving both UI and server code; the tree below is Next.js App Router, and other full-stack frameworks map their routing folder onto `app/` the same way. The **server-client boundary** splits the hexagon: the domain and application DTOs are safe on both sides, use cases and driven adapters run only on the server, and client components reach the server only through driving adapters.

```
app/                             ← routing only: every file is a driving adapter
├── layout.tsx
└── orders/
    ├── page.tsx                 ← calls a use case, renders module components
    └── api/route.ts             ← HTTP driving adapter, for outside callers
src/
├── server/
│   └── composition-root.ts      ← `import "server-only"`
├── shared/                      ← shared kernel, plus shared/ui/
└── modules/ordering/
    ├── index.ts                 ← server interface: use cases (`import "server-only"`)
    ├── domain/                  ← pure, safe on both sides
    │   ├── entities/order.ts
    │   └── ports/order-repository.port.ts
    ├── application/
    │   ├── use-cases/place-order.use-case.ts
    │   └── dtos/                ← safe on both sides
    │       ├── place-order.command.ts
    │       └── order.result.ts
    └── infrastructure/
        ├── driving/
        │   ├── order.actions.ts         ← server actions (`"use server"`)
        │   ├── order-form.tsx           ← client component (`"use client"`)
        │   ├── order-summary.tsx        ← server component
        │   └── dtos/place-order.form.ts
        ├── driven/
        │   ├── prisma-order-repository.adapter.ts
        │   ├── in-memory-order-repository.adapter.ts
        │   └── dtos/prisma-order.model.ts
        └── config/ordering.config.ts
```

- **`app/` holds only routing.** A reserved file (`page.tsx`, `layout.tsx`, `route.ts`, `loading.tsx`) reads params, calls a use case through the module's `index`, and renders the module's components; it holds no rules and no queries.
- **Mark every side.** Server-only files (the module's `index`, use cases, services, driven adapters, wiring, the composition root) import `server-only`; client components open with `"use client"`. The build then fails when a client file pulls in server code.
- **Client components cross the boundary two ways only**: props from a server component, or a server action from `<noun>.actions.ts`. They import the domain and application DTOs freely, and use cases never.
- **Server actions are driving adapters** like routes: validate the form, map it to a command, call one use case, revalidate, return the result.
- **Nuxt, Remix, SvelteKit**: their routing folder (`pages/`, `app/routes/`, `src/routes/`) takes `app/`'s place and rules; their server-only marker (`.server.ts`) takes `server-only`'s.
