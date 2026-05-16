# Module 02 — HTTP Fundamentals: The Language of the Web

**Goal:** Strip away the magic of APIs and look at the raw bytes. Understand precisely how computers talk via the HTTP protocol, how state is managed (or rather, avoided), and how REST maps domain layers conceptually.

## Outcomes

By the end of this module you will be able to:

- Read and interpret raw HTTP requests and responses line-by-line.
- Understand the deep "stateless" nature of HTTP and why it forces specific backend designs.
- Choose the exact correct HTTP methods and status codes for complex edge cases.
- Explain the runtime parsing process of headers, body, and query schemas.

---

## Purpose & system intent (before code)

HTTP is your **contract language**. If you misuse it, clients misbehave, caches lie, and your APIs break in subtle, expensive ways. The goal here is to treat HTTP like a **precision protocol**, not just “some strings.”

## 🧠 Mental Model First: The Post Office

**HTTP is just text sent in the mail.**

Imagine sending a physical package via a post office:

- **The Envelope Front (Request Line & Headers):** Tells the postman where it's going (URL), what shipping priority it is (Method), and metadata like "Handle with care" (Headers).
- **The Box Contents (The Body):** The actual item you are sending (JSON, files, HTML).
- **The Empty Mailbox (Statelessness):** Every single package you send is to an amnesiac postman. They do not remember you from the previous package. If you send 5 letters, you must put your return address (Auth Tokens) on _every single one_.

There is no "permanent magical pipeline" between client and server in HTTP (unlike WebSockets). A request goes out, the thread waits, a response comes back, the connection drops.

---

## 🧭 Visual Explanation: HTTP Lifecycle

### Raw Text of a Request

When you call `fetch("https://api.bun.sh/users")`, the browser translates it into raw ASCII text over a TCP socket.

```http
POST /users HTTP/1.1              <-- 1. Request Line (Method, Path, Protocol)
Host: api.bun.sh                  <-- 2. Header: Where is this going?
Authorization: Bearer xyz123      <-- 2. Header: Who am I?
Content-Type: application/json    <-- 2. Header: How should the server parse the body?
Content-Length: 36                <-- 2. Header: How many bytes to read?
                                  <-- 3. Empty line signifying end of headers
{"name":"Alice","role":"admin"}   <-- 4. The Body (Payload)
```

### Raw Text of a Response

```http
HTTP/1.1 201 Created              <-- 1. Status Line (Protocol, Status Code, Status Text)
Date: Thu, 04 May 2024 12:00:00   <-- 2. Headers
Content-Type: application/json
Connection: keep-alive

{"id":994,"name":"Alice"}         <-- 3. Body
```

### The "REST" Mapping Table

REST (Representational State Transfer) is just an agreement on how to use HTTP verbs predictably.

| HTTP Verb | CRUD Action   | Idempotent? | Explanation & Hidden Complexity                                                      |
| --------- | ------------- | ----------- | ------------------------------------------------------------------------------------ |
| `GET`     | Read          | YES         | Safe to call 1000 times. Browsers heavily cache these. Never use for mutation.       |
| `POST`    | Create        | **NO**      | Calling this 1000 times makes 1000 users. Used for complex actions, too.             |
| `PUT`     | Update (Full) | YES         | Replaces the _entire_ resource. `PUT` with missing fields? Those fields become null. |
| `PATCH`   | Update (Part) | YES         | Only applies changes for fields sent in the body.                                    |
| `DELETE`  | Delete        | YES         | Safely wipes a resource.                                                             |

_(Hidden Complexity: "Idempotency" means doing it once has the same effect as doing it multiple times. If an app crashes during a POST, the browser won't automatically retry it because it might charge your credit card twice. It WILL retry a GET.)_

### Content negotiation (what the client wants)

```text
Client → Accept: application/json
Server → Content-Type: application/json
```

If those mismatch, clients may parse your response incorrectly.

---

## 🔍 Deep Technical Explanation

### 1. Statelessness & Caching

Because HTTP is stateless, backends scale easily horizontally. If Server A dies, Server B can handle the next request because every request contains all the information needed (Token, Body).

### 2. URL Structure

`https://api.domain.com/v1/users?role=admin&sort=desc`

- `https`: Scheme
- `api.domain.com`: Hostname
- `/v1/users`: Path (The "Resource" locator)
- `?role=admin&sort=desc`: Query String (The "Filter/Modifiers")

### 3. Hono's Context (`c`)

In Hono, the `c` parameter you see everywhere (`app.get('/', (c) => ... )`) is a wrapper around the raw incoming HTTP Request and the outgoing Response. It parses strings into JS Objects.

### 4. Cookies & sessions (stateless reality)

HTTP is stateless. Cookies are just headers (`Cookie` in, `Set-Cookie` out). They **do not** live on the server. If you need state, you must store it (DB/Redis) and reference it (token/session id) on every request.

### 5. Hidden complexity

- **Keep-Alive**: One TCP connection can carry multiple requests. Do not assume “one connection = one request.”
- **Chunked transfer**: Body might arrive in chunks; streaming exists.
- **Proxy headers**: `X-Forwarded-For` changes the “real” IP.
- **Caching**: `ETag`, `If-None-Match`, `Cache-Control` decide who serves what.

### TypeScript behavior

- `c.req.query()` returns `string | undefined` → handle missing values.
- JSON bodies are `any` unless validated. Use Zod to avoid runtime crashes.

---

## 🧱 Line-by-Line Code Breakdown

Let's see how we interact with raw HTTP semantics using Hono.

```typescript
import { Hono } from "hono";

const app = new Hono();

// GET: Querying Data
app.get("/search", (c) => {
  // 1. Reading Query Parameters (e.g. ?term=bun&page=2)
  const term = c.req.query("term");
  const page = c.req.query("page") || "1";

  // 2. Reading Headers (e.g. Authorization or custom headers)
  const userAgent = c.req.header("User-Agent");

  // 3. Returning proper Types and Status Codes
  // 200 OK is the default
  return c.json(
    {
      results: [`Found ${term} on page ${page}`],
      agent: userAgent,
    },
    200,
  );
});

// POST: Mutating Data
app.post("/upload", async (c) => {
  // 1. Check Content-Type header. Hono does this automatically via formatting.
  // c.req.json() reads the raw stream of bytes and runs JSON.parse()
  const body = await c.req.json().catch(() => null);

  if (!body) {
    // 400 Bad Request indicates client error (Malformed JSON)
    return c.text("Invalid JSON Payload", 400);
  }

  // 2. Simulating a server failure
  if (body.triggerCrash) {
    // 500 Internal Server Error indicates our fault
    return c.text("Boom!", 500);
  }

  // 3. Setting outgoing Headers (e.g. Rate Limits)
  c.header("X-RateLimit-Remaining", "99");

  // 201 Created is the correct REST response for POST creations
  return c.json({ message: "Processed", data: body }, 201);
});
```

### Execution Flow & Runtime Behavior

1. Bun listens on a TCP port (default 3000).
2. Bun's internal C/Zig HTTP parser reads the raw text bytes arriving over the network interface.
3. Bun converts the raw bytes into a high-performance standard `Request` object.
4. Hono steps in: It takes the `Request` path (`/upload`), traverses its routing tree, and finds the matching controller.
5. `await c.req.json()` reads the memory stream bytes and constructs a JS object.
6. Hono generates a standard `Response` object.
7. Bun takes the `Response`, serializes it back to raw ASCII HTTP-compliant bytes, and pushes it over the TCP socket back to the client.

---

## Debugging mindset 🧪

- If the client hangs, check **Content-Length** vs actual bytes.
- If the client can't parse, check **Content-Type**.
- If caching seems wrong, inspect **Cache-Control/ETag**.

## 🛠 Active Engineering Exercise

**Your Task:**
Use `curl` or Postman to send requests to your server. Look explicitly at the Headers tab.
Run this command in the terminal:
`curl -v -X POST http://localhost:3000/upload -d '{"triggerCrash":true}' -H "Content-Type: application/json"`
Notice the raw arrows `>` (request) and `<` (response) in the terminal output. That is the true language of the web.
