# Module 12 — Error Handling Deep Dive

**Goal:** Build consistent, production-grade error handling that preserves user experience and debugging clarity.

---

## Purpose & system intent (before code)

Errors are **inevitable**. The goal is not to eliminate them—it’s to **contain** them, report them, and respond predictably.

---

## 🧠 Mental Model First: Fire Doors

In a building, fire doors don’t stop fires from starting—they **contain** them. Error handling works the same way.

---

## 🧭 Visual Explanation

```text
Handler throws → Error boundary → Standard response
```

### Error taxonomy

| Type          | Example                | HTTP |
| ------------- | ---------------------- | ---- |
| Validation    | Missing required field | 400  |
| Auth          | Invalid token          | 401  |
| Authorization | Forbidden              | 403  |
| Not Found     | Resource missing       | 404  |
| Conflict      | Duplicate email        | 409  |
| Server        | Unexpected failure     | 500  |

---

## 🔍 Deep Technical Explanation

### Problem+JSON (recommended format)

```json
{
  "type": "https://example.com/errors/validation",
  "title": "Validation Failed",
  "status": 400,
  "detail": "email is required",
  "instance": "/users"
}
```

### Hidden complexity

- **Double responses**: if you catch and rethrow, you might send twice.
- **Leaky stack traces**: never expose internal stack in production.

---

## 🧱 Line-by-Line Code Breakdown (Hono error boundaries)

```ts
import { Hono } from "hono";

const app = new Hono();

app.notFound((c) => c.json({ error: "Not Found" }, 404));

app.onError((err, c) => {
  console.error(err);
  return c.json({ error: "Internal Server Error" }, 500);
});

app.get("/boom", () => {
  throw new Error("Crash");
});
```

### Execution flow

```text
Handler throws → app.onError → 500 response
```

---

## 🛠 Active Engineering Exercise

1. Standardize error responses to a single shape.
2. Add `app.onError` with logging and correlation IDs.
3. Add `app.notFound` handler.

---

## Debugging mindset 🧪

- If error responses vary, centralize in `onError`.
- If a bug is silent, ensure you log **errors and request context**.
