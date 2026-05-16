# Module 08 — Validation Deep Dive

**Goal:** Master validation as a first-class engineering practice: protect the system, keep contracts stable, and produce predictable error responses.

---

## Purpose & system intent (before code)

Validation is **damage control at the border**. If unsafe data enters your system, every downstream layer must “guess” what to do. That’s how bugs and security issues happen.

---

## 🧠 Mental Model First: Customs & Immigration

At an airport, customs checks your paperwork before you enter the country. Your backend must do the same for every request payload.

---

## 🧭 Visual Explanation

```text
Raw Request → Schema Validation → Safe DTO → Business Logic
```

### Validation vs. Types

| Concept          | When it runs | Purpose                   |
| ---------------- | ------------ | ------------------------- |
| TypeScript types | Compile time | Dev safety & autocomplete |
| Zod validation   | Runtime      | Input safety & contracts  |

---

## 🔍 Deep Technical Explanation

### TypeScript behavior (critical)

- TS types are erased at runtime. `req.json()` returns `any`.
- Validation is the **only** way to guarantee runtime shape.

### Hidden complexity

- **Coercion**: user sends `"42"`, you need a number.
- **Partial updates**: PATCH schemas differ from POST schemas.
- **Error structure**: consistent error shapes are part of your API contract.

---

## 🧱 Line-by-Line Code Breakdown

### 1) Define a DTO schema

```ts
import { z } from "zod";

export const CreateUserSchema = z.object({
  email: z.string().email(),
  age: z.coerce.number().int().min(18),
  role: z.enum(["user", "admin"]).default("user"),
});

export type CreateUserDTO = z.infer<typeof CreateUserSchema>;
```

**Line-by-line**

- `z.coerce.number()` lets `"42"` become `42` safely.
- `z.enum(...)` restricts values to a known set.
- `z.infer` gives you a TS type from the runtime schema.

### 2) Use schema in a route

```ts
import { Hono } from "hono";
import { CreateUserSchema } from "./schema";

const app = new Hono();

app.post("/users", async (c) => {
  try {
    const raw = await c.req.json();
    const dto = CreateUserSchema.parse(raw);
    return c.json({ data: dto }, 201);
  } catch (err: any) {
    return c.json({ error: "Validation Failed", details: err.errors }, 400);
  }
});
```

### Execution flow

```text
Request → JSON parse → Zod parse → DTO → Service
```

---

## 🛠 Active Engineering Exercise

1. Create a `PATCH /users/:id` schema that allows **optional** fields.
2. Enforce **string trimming** and **minimum length** on a `username` field.
3. Ensure errors return in the format:

```json
{ "error": "Validation Failed", "details": [ ... ] }
```

---

## Debugging mindset 🧪

- If validation fails unexpectedly, log the **raw input** (not the parsed DTO).
- If `age` is `NaN`, check coercion rules.
