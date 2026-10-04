# API Design

## Overview

Design network endpoints (REST, GraphQL) that are predictable to consume and safe to retry. Every API is an interface, so read [INTERFACE.md](../interface/INTERFACE.md) first: its contract-first, additive-change, and boundary-validation principles apply here and are not repeated.

## When to Use

- Designing new API endpoints
- Changing existing public endpoints
- Establishing database schema that informs API shape
- Any state-changing endpoint that a client or queue may retry

## When Not to Use

- Idempotency keys on read-only endpoints; they belong on state-changing ones that a client or queue may retry

## Core Principles

### 1. Consistent Error Semantics

Pick one error strategy and use it everywhere:

```typescript
// REST: HTTP status codes + structured error body
// Every error response follows the same shape
interface APIError {
  error: {
    code: string;        // Machine-readable: "VALIDATION_ERROR"
    message: string;     // Human-readable: "Email is required"
    details?: unknown;   // Additional context when helpful
  };
}

// Status code mapping
// 400 → Client sent invalid data
// 401 → Not authenticated
// 403 → Authenticated but not authorized
// 404 → Resource not found
// 409 → Conflict (duplicate, version mismatch)
// 422 → Validation failed (semantically invalid)
// 500 → Server error (never expose internal details)
```

**Don't mix patterns.** If some endpoints throw, others return null, and others return `{ error }` — the consumer can't predict behavior.

Validation at the boundary maps onto that shape:

```typescript
app.post('/api/tasks', async (req, res) => {
  const result = CreateTaskSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Invalid task data',
        details: result.error.flatten(),
      },
    });
  }

  // After validation, internal code trusts the types
  const task = await taskService.create(result.data);
  return res.status(201).json(task);
});
```

### 2. Predictable Naming

| Pattern | Convention | Example |
|---------|-----------|---------|
| REST endpoints | Plural nouns, no verbs | `GET /api/tasks`, `POST /api/tasks` |
| Query params | camelCase | `?sortBy=createdAt&pageSize=20` |
| Response fields | camelCase | `{ createdAt, updatedAt, taskId }` |
| Boolean fields | is/has/can prefix | `isComplete`, `hasAttachments` |
| Enum values | UPPER_SNAKE | `"IN_PROGRESS"`, `"COMPLETED"` |

### 3. Honouring an Idempotency Key

Accepting an `Idempotency-Key` is the contract. Honouring it is the implementation, and it is where the money is lost — a key the server accepts but handles carelessly is worse than no key at all, because the client now believes retrying is safe.

**Derive the key from the intent, not the attempt.** The key must be stable across retries of one intent and different across distinct intents:

```typescript
crypto.randomUUID()                    // ✗ new key per attempt — every retry is a new charge
`${userId}:${amount}`                  // ✗ two legitimate $50 charges collapse into one
`${orderId}:${Date.now()}`             // ✗ a timestamp is randomUUID() wearing a hat

req.headers['idempotency-key']         // ✓ client generates once, reuses on retry
`charge:v1:${orderId}`                 // ✓ derived from an immutable identifier
```

The key comes from the client or the initiating event — never from the layer doing the retrying.

**Claim atomically. A check followed by an act is a race:**

```typescript
// ✗ TOCTOU: two concurrent retries both read "not seen", both charge
if (!(await db.exists(key))) {
  await chargeCard(amount);
  await db.insert(key);
}

// ✓ let the unique constraint pick the winner
try {
  await db.insert({ key, state: 'in_progress', requestHash });
} catch (e) {
  if (isUniqueViolation(e)) return replayOrReject(key);
  throw;
}
const result = await chargeCard(amount);
await db.update({ key, state: 'succeeded', response: result });
```

The unique constraint *is* the mechanism. A store that cannot enforce uniqueness in one operation cannot back this.

**Guard the payload.** Same key with a different body is a client bug, and must fail loudly rather than serving the first response to a second request:

```typescript
if (existing.requestHash !== hash(req.body)) {
  return res.status(422).json({ error: 'idempotency key reused with a different payload' });
}
```

**Decide what an in-flight duplicate gets.** The first request is still running when the second arrives — the common case under retry storms:

| Strategy | Response | Use when |
|---|---|---|
| Reject | `409 Conflict` | Client can retry later; simplest and safest |
| Wait | Block for the result, bounded | Caller needs it synchronously |
| Return pending | `202` + status URL | Long-running effects |

Never let the second caller through because the first "seems stuck". A stalled attempt whose fate is unknown is exactly when duplicating costs most.

**Every call has three outcomes, not two: success, failure, and _unknown_.** A timeout tells you nothing about whether the effect applied. Record the intent *before* calling out, so a crash between the call and the response leaves evidence something must resolve later — rather than a silently retried charge.

**Set retention from the longest retry chain**, not from disk cost. Keys must outlive every path that can re-deliver the same intent, including a dead-letter queue replayed a week later and any provider dispute window. A 24-hour key TTL behind a 7-day DLQ is a duplicate waiting to happen.

## REST API Patterns

### Resource Design

```
GET    /api/tasks              → List tasks (with query params for filtering)
POST   /api/tasks              → Create a task
GET    /api/tasks/:id          → Get a single task
PATCH  /api/tasks/:id          → Update a task (partial)
DELETE /api/tasks/:id          → Delete a task

GET    /api/tasks/:id/comments → List comments for a task (sub-resource)
POST   /api/tasks/:id/comments → Add a comment to a task
```

### Pagination

Paginate list endpoints:

```typescript
// Request
GET /api/tasks?page=1&pageSize=20&sortBy=createdAt&sortOrder=desc

// Response
{
  "data": [...],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 142,
    "totalPages": 8
  }
}
```

### Filtering

Use query parameters for filters:

```
GET /api/tasks?status=in_progress&assignee=user123&createdAfter=2025-01-01
```

### Partial Updates (PATCH)

Accept partial objects — only update what's provided:

```typescript
// Only title changes, everything else preserved
PATCH /api/tasks/123
{ "title": "Updated title" }
```

## Common Rationalizations

| Rationalization | Reality |
|---|---|
| "We don't need pagination for now" | You will the moment someone has 100+ items. Add it from the start. |
| "PATCH is complicated, let's just use PUT" | PUT requires the full object every time. PATCH is what clients actually want. |
| "Accepting the Idempotency-Key header is enough" | The header is the contract; storing the key against the result is the implementation. A key you accept but don't honour tells the client retrying is safe when it isn't. |
| "Our queue guarantees exactly-once delivery" | No queue does across a consumer crash — the broker's ack and your side effect are not in one transaction. Design for at-least-once with idempotent processing. |
| "Duplicate requests are rare" | They're *correlated*. Retries spike exactly when a dependency is degraded — the moment duplicates are most likely and most expensive. |

## Verification

After designing an API:

- [ ] Interface verification in [INTERFACE.md](../interface/INTERFACE.md) passes
- [ ] URLs name resources, never verbs (`/tasks`, not `/api/createTask`)
- [ ] Error responses follow a single consistent format
- [ ] List endpoints support pagination
- [ ] Naming follows consistent conventions across all endpoints
- [ ] State-changing endpoints either honour an idempotency key or are documented as unsafe to retry
- [ ] The key is claimed in one atomic operation, guarded by a unique constraint (a `SELECT` then `INSERT` is a race)
- [ ] The key identifies the request across attempts, never a UUID or timestamp regenerated per attempt
- [ ] A reused key with a different payload fails loudly rather than replaying the wrong response
- [ ] The in-flight-duplicate response is a deliberate choice (409, wait, or 202) rather than whatever falls out
- [ ] Key retention outlives the longest retry path, including dead-letter replay
