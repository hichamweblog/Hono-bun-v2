# Module 06 — Request & Response Handling

**Goal:** Understand how to precisely extract complex data from incoming requests and craft perfect, standards-compliant JSON/HTML/Stream responses.

## Outcomes

By the end of this module you will be able to:

- Confidently parse URL Parameters, Query Strings, and HTTP Headers.
- Parse JSON, Form-Data, and Multipart bodies using native standard APIs.
- Utilize Hono primitives to generate precise HTTP standard responses.
- Construct semantic HTTP Status codes.

---

## Purpose & system intent (before code)

Request/response handling is your **input/output control panel**. If you parse the wrong fields or return the wrong headers, everything downstream breaks—clients, caches, and monitoring.

## 🧠 Mental Model First: The Bouncer and the Chef

**Requests (The Bouncer)**
Incoming data is dirty and chaotic. The framework acts as a bouncer, patting down the incoming request to find what you need. Are the keys in the query string? Is the wallet in the headers? Is the heavy luggage in the JSON body? You must know where to look.

**Responses (The Chef)**
After business logic executes, you are a chef plating the food. If you serve soup on a flat plate, it's useless. You must present the data exactly how the client expects it (Headers + Content-Type + Status Code).

---

## 🧭 Visual Explanation: Where is my data?

```text
POST /users/55/details?region=US&darkMode=true HTTP/1.1
Host: api.bun.sh
Authorization: Bearer 123
Content-Type: application/json

{"email": "test@test.com"}
```

| Location         | Hono Syntax                       | What is it for?                 |
| ---------------- | --------------------------------- | ------------------------------- |
| **Path Params**  | `c.req.param('id')` -> `"55"`     | The core resource ID.           |
| **Query Params** | `c.req.query('region')` -> `"US"` | Filters, sorting, metadata.     |
| **Headers**      | `c.req.header('Authorization')`   | Auth, Rate Limits, Client Info. |
| **Body (JSON)**  | `await c.req.json()`              | Complex nested data schemas.    |
| **Body (Form)**  | `await c.req.formData()`          | File uploads, legacy forms.     |

---

## 🧱 Line-by-Line Code Breakdown

```typescript
import { Hono } from "hono";

const app = new Hono();

app.post("/profile/:id", async (c) => {
  // 1. Path Params (Always Strings!)
  const userId = c.req.param("id");

  // 2. Query Params
  // Returns string | undefined depending on if it exists.
  const theme = c.req.query("darkMode");

  // 3. Headers
  const token = c.req.header("Authorization");

  // 4. Body Parsing
  // WARNING: If the client sends malformed JSON, this will throw an exception!
  // Production apps wrap this in a try/catch or Zod Validator.
  const body = await c.req.json();

  // 5. Advanced: Getting raw Request object
  // If Hono doesn't have a helper, drop down to standard Web API.
  const rawUrl = c.req.url;

  // ----------------------------------------------------
  // RESPONSE BUILDING (Plating the food)

  // Method 1: The standard JSON response (Automatic 200 OK & Application/Json header)
  // return c.json({ status: "success" })

  // Method 2: Setting a specific HTTP status code (201 Created)
  // return c.json({ data: body }, 201)

  // Method 3: Returning raw text
  // return c.text("Done!")

  // Method 4: Modifying headers before responding
  c.header("X-Custom-Time", Date.now().toString());

  return c.json(
    {
      user: userId,
      receivedTheme: theme,
      payload: body,
    },
    200,
  );
});
```

### Execution flow (runtime)

```text
Request → Param parse → Query parse → Header read → Body parse → Response
```

### TypeScript behavior

- `c.req.query()` returns `string | undefined`.
- `c.req.param()` returns `string` even for numeric IDs.
- `await c.req.json()` returns `any` unless validated.

### Hidden complexity

- **Malformed JSON** throws, so wrap in `try/catch` or validation.
- **Multipart** bodies can be huge; avoid buffering all in memory.
- **Content-Type mismatch** leads to parsing errors.

## 🛠 Active Engineering Exercise

**Your Task:**

1. Create a `POST /echo/:id` route.
2. Inside it, extract an `id` param, a `?search=` query, an `X-Api-Key` header, and a JSON body.
3. Return all 4 of those variables mixed together cleanly inside a single JSON response payload, along with a `202 Created` status code.

---

## Debugging mindset 🧪

- If data is `undefined`, check **where it lives** (param vs query vs header vs body).
- If the body parse fails, inspect **Content-Type** and raw payload.
