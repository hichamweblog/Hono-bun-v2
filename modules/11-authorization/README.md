# Module 11 — Authorization Deep Dive

**Goal:** Learn how to enforce **who can do what** with clean policies, consistent errors, and maintainable role/permission models.

---

## Purpose & system intent (before code)

Authentication says **who you are**. Authorization says **what you are allowed to do**. A secure backend must never mix them.

---

## 🧠 Mental Model First: The Concert Wristbands

Everyone shows a ticket at the gate (auth). But only VIP wristbands allow backstage access (authorization).

---

## 🧭 Visual Explanation

```text
Request → Auth (identify) → Authorize (policy check) → Handler
```

### Authorization models

| Model | Example                 | Good for             |
| ----- | ----------------------- | -------------------- |
| RBAC  | role = admin            | Simple orgs          |
| ABAC  | role + ownership + time | Complex policies     |
| PBAC  | explicit permissions    | Fine‑grained control |

---

## 🔍 Deep Technical Explanation

### Hidden complexity

- **Ownership checks**: user can edit _their own_ record, not others.
- **Resource hierarchy**: team → project → task permissions.
- **Policy drift**: inconsistent checks across endpoints cause security gaps.

### TypeScript behavior

- Represent permissions as **string literal unions** to avoid typos.
- Use `as const` arrays for type-safe permission lists.

---

## 🧱 Line-by-Line Code Breakdown (Policy middleware)

```ts
import { Hono } from "hono";

const app = new Hono();

type Permission = "user:read" | "user:write" | "admin:all";

function requirePermission(permission: Permission) {
  return async (c: any, next: any) => {
    const user = c.get("user");
    if (!user?.permissions?.includes(permission)) {
      return c.json({ error: "Forbidden" }, 403);
    }
    await next();
  };
}

app.get("/users", requirePermission("user:read"), async (c) => {
  return c.json([{ id: 1, email: "demo@site.com" }]);
});
```

### Execution flow

```text
Auth → Set user → requirePermission → Handler
```

---

## 🛠 Active Engineering Exercise

1. Add `role` to your user model.
2. Create an admin-only route.
3. Add a resource ownership check (`/users/:id`).

---

## Debugging mindset 🧪

- If access leaks, search for endpoints **missing policy checks**.
- If users are blocked, inspect **permission arrays** and case sensitivity.
