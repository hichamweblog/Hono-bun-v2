# Module 07 — Middleware Deep Dive

**Goal:** Build a production-grade mental model for middleware, understand execution flow, and learn patterns that keep code safe, testable, and fast.

---

## Purpose & system intent (before code)

Middleware is the **control plane** of your API. It’s where you enforce **security**, **observability**, **rate limits**, and **request hygiene** _before_ business logic runs. Without it, your backend becomes a wide-open door.

---

## 🧠 Mental Model First: Airport Security Conveyor Belts

Requests are luggage. Middleware is the conveyor belt + scanners. If something is unsafe, it never reaches the gate.

---

## 🧭 Visual Explanation

```text
Request
  │
  ├─> Logger (timestamps, tracing)
  ├─> Auth (who are you?)
  ├─> Validation (is payload safe?)
  ├─> Rate Limit (are you abusive?)
  └─> Handler (business logic)
  │
Response
```

### Middleware timeline (before/after)

```text
Before → await next() → After
```

---

## 🔍 Deep Technical Explanation

### Execution flow (important)

```text
Incoming Request
  │
  ▼
Middleware A (before)
  │
  ▼
Middleware B (before)
  │
  ▼
Route Handler
  │
  ▲
Middleware B (after)
  │
  ▲
Middleware A (after)
  │
  ▼
Response
```

### TypeScript behavior

- Use `Variables` generics so middleware can set values safely.
- If you don’t type `Variables`, `c.get()` becomes `unknown` and you lose safety.

### Hidden complexity

- **Missing `await next()`** → the request hangs or returns early.
- **Multiple responses** → if a middleware returns a response, downstream handlers should not run.
- **Error boundaries** → errors should be transformed into consistent HTTP responses.

---

## 🧱 Line-by-Line Code Breakdown

### 1) Timing middleware (observability)

```ts
import { Hono } from "hono";

type Variables = {
  requestId: string;
};

const app = new Hono<{ Variables: Variables }>();

app.use("*", async (c, next) => {
  // 1. Create a request id (traceability)
  const requestId = crypto.randomUUID();
  c.set("requestId", requestId);

  // 2. Start timer
  const start = performance.now();

  // 3. Run downstream handlers
  await next();

  // 4. Add timing headers after handler finishes
  const ms = Math.round(performance.now() - start);
  c.header("X-Response-Time", `${ms}ms`);
  c.header("X-Request-Id", requestId);
});
```

### 2) Auth middleware (early rejection)

```ts
app.use("/private/*", async (c, next) => {
  const token = c.req.header("Authorization");
  if (!token) {
    return c.json({ error: "Missing token" }, 401);
  }
  await next();
});
```

### Line-by-line explanation (auth middleware)

- `app.use("/private/*"...)`: only protects routes under `/private`.
- `c.req.header("Authorization")`: reads the header without parsing the body.
- `return c.json(..., 401)`: terminates the chain early.
- `await next()`: continues only if authorized.

---

## 🛠 Active Engineering Exercise

**Your Task:**

1. Add a logging middleware that prints method, path, and duration.
2. Add an auth middleware that protects `/admin/*` routes.
3. Add a middleware that injects `X-App-Version` into every response.

---

## Debugging mindset 🧪

- If a handler never runs, check for **early returns** in middleware.
- If a header is missing, check that it’s set **after** `await next()`.
- If `c.get()` is `undefined`, confirm the middleware ran and used the correct key.
