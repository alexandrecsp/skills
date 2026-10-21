# Frontend

A UI with no server of its own, talking to APIs over the network. The UI is the driving side; the APIs, storage and browser services are the driven side.

```
src/
├── main.tsx                     ← mounts the app
├── app/                         ← the shell: router, providers, composition root
│   ├── router.tsx
│   └── composition-root.ts
├── shared/                      ← shared kernel, plus shared/ui/ for design-system components
└── modules/cart/
    ├── index.ts                 ← the module's interface: its routes, use cases and application DTOs
    ├── domain/
    │   ├── entities/cart.ts
    │   └── ports/cart-api.port.ts
    ├── application/
    │   ├── use-cases/add-to-cart.use-case.ts
    │   └── dtos/cart.result.ts
    └── infrastructure/
        ├── driving/
        │   ├── cart-page.tsx
        │   ├── cart-line.tsx
        │   ├── use-cart.ts              ← hook: binds components to use cases
        │   └── dtos/add-to-cart.form.ts ← form values
        └── driven/
            ├── http-cart-api.adapter.ts
            ├── in-memory-cart-api.adapter.ts
            └── dtos/http-cart.payload.ts
```

- **Components are driving adapters.** They render results and turn events into use-case calls, through a hook named `use-<noun>.ts`. Rules in a component belong in the domain, where tests reach them without rendering.
- **Components and hooks keep their framework's names** (`cart-line.tsx` exports `CartLine`, `use-cart.ts` exports `useCart`), and form values take the `Form` suffix: the folder already names them driving adapters.
- **Server state** (a query cache such as TanStack Query) lives in the driving hook; the query function calls a use case, never `fetch` inline.
- **Route files** a file-based router reserves keep their framework names in `app/` and import from the module's `index`.
