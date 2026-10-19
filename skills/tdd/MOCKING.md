Mock only what you don't control or can't run in the test:

- External APIs (payment, email)
- Time and randomness
- Databases and the file system, when no local stand-in exists (prefer a test database or an in-memory filesystem)

Everything you control, your own modules and their internal collaborators, runs for real in the test.

## Designing a mocked dependency

Inject each mocked dependency rather than creating it inside, as the `codebase-design` skill's testability rules describe. Then give it an **SDK-style interface**: one function per external operation, rather than one generic fetcher.

```typescript
// GOOD: each function is independently mockable
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// BAD: the mock needs conditional logic to answer each endpoint
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

With the SDK style, each mock returns one specific shape, test setup carries no conditional logic, each test shows which endpoints it exercises, and each endpoint keeps its own types.
