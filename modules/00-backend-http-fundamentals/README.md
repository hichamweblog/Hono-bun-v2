# Module 00 — Backend & HTTP Fundamentals

**Goal:** Build a _mental model_ of how backend systems and HTTP requests actually work, then implement a minimal HTTP server in Bun. This is the foundation for every advanced topic later.

## Outcomes

By the end of this module you should be able to:

- Explain the backend’s role in a system (and what it _is not_).
- Describe the HTTP request lifecycle end-to-end.
- Choose correct HTTP methods and status codes for common actions.
- Build a minimal Bun HTTP server and reason about its behavior.
- Identify common HTTP mistakes and debug them effectively.

---

## Purpose & system intent (before code)

If you can’t **name the purpose** of a backend in one sentence, you’ll build features that are hard to secure, hard to scale, and impossible to debug.

**Purpose of a backend:**

- **Enforce rules** (who can do what, and under which conditions).
- **Protect data** (validate, sanitize, authorize, audit).
- **Coordinate work** (DB, cache, queues, external APIs).
- **Speak a contract** (consistent HTTP semantics).

**A backend is _not_:**

- “Just a JSON server.”
- A place to “dump logic.”
- A giant file of endpoints.

---

## Conceptual explanation

Backend systems are **decision engines** that enforce rules and protect data. HTTP is the standardized envelope for those decisions. This module builds the foundation for every routing, validation, and data layer you’ll use later.

## Mental model first 🧠

**Backend is the kitchen, not the menu.**

- The **frontend** is the menu and dining room (what the user sees).
- The **backend** is the kitchen: it handles orders, validates ingredients, and ensures quality.

**HTTP is the order slip.**

- Each request is a single order slip.
- The backend must **interpret** it, **verify** it, **fulfill** it, and **respond**.

If you treat the backend as “just a server,” you’ll build brittle systems. If you treat it as a _process with responsibility and contracts_, you’ll build reliable ones.

---

## Visual explanation first 🧭

### Request lifecycle (high-level)

```txt
Client
  │  (HTTP Request)
  ▼
Runtime (Bun)
  │
  ▼
Your Server Code
  │
  ▼
Response Builder
  │  (HTTP Response)
  ▼
Client
```

### What a single request really contains

```txt
Request Line:  METHOD  PATH  VERSION
Headers:       Key: Value
Body:          Raw bytes (optional)
```

### Typical backend responsibility layers

```txt
Transport (HTTP)
   ▼
Routing (map URL + method → handler)
   ▼
Validation (is input safe + correct?)
   ▼
Business Logic (rules + decisions)
   ▼
Data Access (DB/cache/files)
   ▼
Response (status + body + headers)
```

### Execution flow (runtime view)

```txt
TCP/TLS Socket
  │  (raw bytes)
  ▼
HTTP Parser (Bun)
  │  (Request object)
  ▼
Router (Hono / your handler)
  │  (decide path + method)
  ▼
Handler
  │  (logic + I/O)
  ▼
Response
  │  (status + headers + body)
  ▼
Serializer → Bytes → Client
```

---

## Deep technical explanation 🔍

### Why HTTP exists

HTTP is a standardized, text-based protocol for moving messages across a network. Standardization lets any client (browser, mobile, CLI) talk to any server using the same rules.

### HTTP is _stateless_

Each request is isolated. If you need memory between requests, **you must create it** (sessions, tokens, DB state, cache). This is both a strength (simplicity) and a danger (bugs when you assume memory).

### Methods are intent, not just verbs

- **GET** → read data (safe, idempotent)
- **POST** → create/change with side effects (not idempotent)
- **PUT** → replace fully (idempotent)
- **PATCH** → partial update (idempotent _if designed that way_)
- **DELETE** → remove (idempotent if deleting the same resource repeatedly)

### Status codes are _contracts_

Status codes aren’t “feelings.” They tell clients how to interpret the response.

- **2xx** success
- **3xx** redirect
- **4xx** client error (your fault)
- **5xx** server error (my fault)

### Headers are metadata and control

Headers control caching, authentication, CORS, content type, and more. They’re critical for production behavior—don’t treat them as optional.

### Performance implications

Every request costs:

- Network latency
- Parsing + routing
- Serialization/deserialization
- DB or cache I/O
  You don’t optimize the handler first; you **optimize the slowest layer**.

### Security implications

Input is hostile by default. If you don’t validate inputs, you are building an exploit surface, not a system.

### TypeScript behavior (critical)

- The Web `Request` and `Response` are **standard types**. Bun and Hono use them directly.
- `await req.json()` returns `any` by default. That means **TypeScript will not protect you** unless you validate.
- Use Zod or a parser before using the data. Validation is _runtime safety_; types are _compile-time hints_.

### Hidden complexity (what bites in production)

- **Concurrent requests** can read or mutate shared state at the same time → race conditions.
- **Timeouts** cause partial operations (DB write may succeed but response times out).
- **Body parsing** can fail due to invalid JSON or huge payloads → must guard size & parse errors.
- **Client disconnects** may cancel the request → use `AbortSignal` when doing expensive work.

---

## Step-by-step coding (Guided → Independent) 🧱

We’ll build a minimal Bun HTTP server to make the lifecycle real.

### Step 1 — Create a tiny server (fully guided)

Create a file `index.ts` and add a server that responds with plain text:

```ts
type HandlerResult = Response | Promise<Response>;

function handle(req: Request): HandlerResult {
  // 1) Parse the URL once (avoid double parsing in real apps)
  const url = new URL(req.url);

  // 2) Route: GET /
  if (req.method === "GET" && url.pathname === "/") {
    return new Response("Hello from Bun", {
      status: 200,
      headers: { "content-type": "text/plain" },
    });
  }

  // 3) Fallback
  return new Response("Not Found", { status: 404 });
}

const server = Bun.serve({
  port: 3000,
  fetch: handle, // Bun calls this for every request
});

console.log(`Listening on ${server.url}`);
```

### Line-by-line explanation

- **`type HandlerResult`**: shows you can return a `Response` or a Promise of one.
- **`new URL(req.url)`**: parses the request URL once; parsing is not free.
- **`req.method` + `url.pathname`**: your routing gate; correctness starts here.
- **`new Response(...)`**: constructs the actual HTTP response object.
- **`Bun.serve({ fetch })`**: registers your handler with Bun’s HTTP server.

### Step 2 — Add a JSON endpoint (guided)

Add a `/health` endpoint that returns JSON:

```ts
if (req.method === "GET" && new URL(req.url).pathname === "/health") {
  return Response.json(
    { status: "ok", timestamp: new Date().toISOString() },
    { status: 200 },
  );
}
```

### Step 3 — Parse JSON input safely (partial guidance)

Add a `/echo` endpoint that accepts JSON and returns it back:

```ts
if (/* method + path check */) {
  try {
    const data = await req.json();
    // TODO: return a JSON response with { received: data }
  } catch {
    // TODO: return a 400 with a safe error body
  }
}
```

### Step 4 — Add method discipline (challenge)

If `/echo` is hit with a non‑POST method, return **405 Method Not Allowed**.

**Hint:** check the path and the method before parsing the body.

---

## Best practices you should internalize ✅

- **Be explicit** about method + path handling.
- **Return correct status codes**—they’re part of your API contract.
- **Use JSON consistently** (don’t mix JSON and text in the same API unless intentional).
- **Validate early** and fail fast.
- **Separate routing from logic** (we’ll formalize this in later modules).

---

## Common mistakes (and why they hurt) ⚠️

- **Returning 200 on errors** → clients can’t detect failure.
- **Parsing JSON without try/catch** → crashes the request path.
- **Ignoring methods** → accidental mutating operations via GET.
- **Using global mutable state** → breaks under concurrent requests.

---

## Debugging mindset 🧪

When a request fails, ask:

1. **Did the request reach the server?** (Check logs)
2. **Was the handler matched?** (Path/method mismatch?)
3. **Did parsing fail?** (Invalid JSON? missing headers?)
4. **Did you return a Response in every branch?**

**Pro tip:** the fastest way to debug HTTP is to _inspect the raw request_ and compare to what your code expects.

### Fast debugging checklist

- Does the request match **method + path**?
- Did you **parse** the body safely?
- Are you returning **exact status codes** and `Content-Type`?
- Are you relying on **global state** that might be mutated by other requests?

---

## Exercises (don’t skip) 🏋️

### Exercise A — Status code discipline

Add a `/status` route that:

- Accepts a query param `code` (e.g., `/status?code=418`)
- Returns a response with that status code

**Hint:** query params live on `URL.searchParams`.

### Exercise B — Method safety

Add a `/notes` endpoint:

- `GET /notes` returns a stub list
- `POST /notes` returns `201 Created` with a stub object
- Any other method returns `405`

### Exercise C — Error contracts

If JSON parsing fails, return:

```json
{ "error": { "code": "INVALID_JSON", "message": "Body must be valid JSON" } }
```

Do not include a stack trace.

---

## Mini challenges 🧩

1. **Design question:** When should you use `PUT` vs `PATCH`? Give a real-world example.
2. **Architecture question:** Why is returning `404` for unknown routes safer than returning `200` with an error message?
3. **Performance question:** Which is usually slower—JSON parsing or DB access? Why?

---

## Real-world production notes 🧯

- In production, your server usually sits **behind a reverse proxy** (Nginx, Caddy, Cloudflare, etc.).
- TLS is almost always terminated _before_ your app, but you still must respect `X-Forwarded-*` headers.
- Timeouts matter. A request stuck for 60 seconds can melt your server.

---

## What I need from you next

Reply with:

1. Your answers to the **mini challenges**.
2. Which exercise you want reviewed first.
3. Any confusion about HTTP you want clarified.

Then we’ll move to Module 01.
