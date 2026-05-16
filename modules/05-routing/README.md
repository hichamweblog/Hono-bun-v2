# Module 05 — Routing & Organization

**Goal:** Transform a messy single-file API into a cleanly organized, scalable routing tree using Hono's sub-routers and grouping techniques.

## Outcomes

By the end of this module you will be able to:

- Extract logical domains into separate sub-routers.
- Chain and mount routers to create a RESTful path hierarchy.
- Utilize path parameters and wildcards correctly.
- Understand the hidden complexity of route matching precedence.

---

## Purpose & system intent (before code)

Routing is how you **encode your domain** into a URL map. Good routing makes APIs predictable and debuggable; bad routing makes your system feel random. This module teaches you to design **stable, scalable route trees**.

## 🧠 Mental Model First: The Corporate Office Directory

Imagine walking into a massive corporate headquarters.
If the entire company was jammed into one giant bullpen, finding "Dave from IT Accounting" would be impossible.

Instead, the building has structure:

- **Base Level:** The Front Desk (The Main App Router).
- **Floor 3:** The HR Department (`/hr` Sub-router).
- **Floor 4:** The Tech Department (`/tech` Sub-router).
  Within Floor 4, there are specific desks:
- Desk 1: DevOps (`/tech/devops`)
- Desk 2: Engineering (`/tech/engineering`)

Your API must follow this organizational structure. A 5,000-line `index.ts` file acts like the single messy bullpen. **Sub-routers** give your codebase structured "floors" and "desks" so a developer always knows exactly where to look.

---

## 🧭 Visual Explanation: The Route Tree

```text
index.ts (App Base) `new Hono()`
├── /api
│   ├── /v1
│   │   ├── /users      -> mounted from `userRouter.ts`
│   │   │   ├── GET /          (Get all)
│   │   │   ├── POST /         (Create)
│   │   │   └── GET /:id       (Get specific)
│   │   ├── /teams      -> mounted from `teamRouter.ts`
│   │   │   └── GET /
```

### Table: URL Segment Philosophy

| Segment Type     | Syntax         | Purpose                      | Example         |
| ---------------- | -------------- | ---------------------------- | --------------- |
| Static           | `/users`       | Fixed resources              | `GET /users`    |
| Variable         | `/:id`         | Dynamic identifiers          | `GET /users/99` |
| Wildcard         | `/*`           | Catch-all (usually for 404s) | `ALL /*`        |
| RegEx Restricted | `/:id{[0-9]+}` | Enforce exact shapes         | `GET /users/55` |

---

## 🔍 Deep Technical Explanation

### Why Grouping Matters

At compile/boot time, Hono merges all your sub-routers into a single flat Trie structure. This means organizing your router files into 15 different files **does not slow down execution speed**. The execution speed is exactly the same as if it were all in one file, but the **developer speed** increases drastically.

### Route Precedence (Hidden Complexity)

What happens if you have these two routes?

1. `/users/admin`
2. `/users/:id`

If someone requests `/users/admin`, does it trigger route 1, or does it trigger route 2 where `id = "admin"`?
Unlike old frameworks where the order of file definitions matter, Hono's Trie router is smart enough to score **static paths higher than dynamic parameters**. Route 1 will safely match first.

### Execution flow

```text
Request → Router Trie → Param extraction → Handler
```

### TypeScript behavior

- `c.req.param('id')` is always a **string**. Convert with `Number()` when needed.
- Use `z.coerce.number()` to convert and validate safely.

### Hidden complexity

- **Overlapping routes**: `/users/:id` can mask `/users/admin` if precedence is wrong.
- **Wildcard routes** (`/*`) can shadow deeper paths.
- **Versioning**: `/v1` vs `/v2` should be explicit, not implicit.

---

## 🧱 Line-by-Line Code Breakdown

Let's build a clean, multi-file modular API.

### 1. The Sub-Router (The "Floor")

Create a file strictly dedicated to Users.

```typescript
// routes/users.ts
import { Hono } from "hono";

// 1. Create a specialized Hono instance just for this domain
export const userApp = new Hono();

// 2. Define routes relative to the sub-router's base path
// IMPORTANT: Notice this is just `/`, NOT `/users`
userApp.get("/", (c) => {
  return c.json([{ name: "Alice" }]);
});

// 3. Dynamic parameter routing
userApp.get("/:id", (c) => {
  // Extracting the parameter from the URL string
  const id = c.req.param("id");

  return c.json({ id, message: `Details for user ${id}` });
});

// 4. RegEx restricted routing
userApp.delete("/:id{[0-9]+}", (c) => {
  // This route will ONLY fire if ID is made of numbers.
  // /users/99 -> executes
  // /users/abc -> 404 Not Found
  const id = c.req.param("id");
  return c.text(`Deleted ${id}`);
});
```

### 2. The Main Root (The "Lobby")

Wire the sub-routers together into the main execution entrypoint.

```typescript
// index.ts
import { Hono } from "hono";
import { userApp } from "./routes/users";

const app = new Hono();

// Mount the userApp onto the `/users` path
app.route("/users", userApp);

// Provide a global 404 fallback
app.notFound((c) => {
  return c.json({ error: "Route not found" }, 404);
});

export default app;
```

## 🛠 Active Engineering Exercise

**Your Task:**

1. Inside your workspace, create a `src/routes/` directory.
2. Create `posts.ts` and `comments.ts` sub-routers.
3. In `index.ts`, mount them to `/posts` and `/comments`.
4. Test that `GET /posts/123/comments` can be structured elegantly by nesting routers or using deep path naming conventions!

---

## Debugging mindset 🧪

- If a route doesn’t match, check **mount path + local path**.
- Inspect raw routes with a quick log at startup.
