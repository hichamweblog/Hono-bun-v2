# Module 10 — Authentication Deep Dive

**Goal:** Build a production-ready mental model for authentication: identity verification, sessions, tokens, and secure storage of credentials.

---

## Purpose & system intent (before code)

Authentication answers **“Who are you?”** If this is wrong, every downstream security decision collapses.

---

## 🧠 Mental Model First: The Hotel Check‑In

You show ID at the front desk (login). The hotel gives you a keycard (session/JWT). Every time you enter a room, the keycard proves you’re allowed in.

---

## 🧭 Visual Explanation

### Session-based auth flow

```text
Login → Server creates session → Session ID cookie → Subsequent requests use cookie
```

### JWT-based auth flow

```text
Login → Server signs token → Client stores token → Requests send Bearer token
```

### Sessions vs JWTs (trade‑offs)

| Approach               | Pros                             | Cons                                  |
| ---------------------- | -------------------------------- | ------------------------------------- |
| Session (server state) | Easy revocation, simple          | Needs server storage                  |
| JWT (stateless)        | Scales easily, no server storage | Harder revocation, token leakage risk |

---

## 🔍 Deep Technical Explanation

### Password storage (non‑negotiable)

- **Never store raw passwords.** Use Argon2 or bcrypt.
- Store `password_hash` only. Validate by hashing the incoming password and comparing.

### Refresh tokens

- Short‑lived access token + long‑lived refresh token = safety + usability.
- Refresh token should be stored securely (HTTP‑only cookie).

### Hidden complexity

- **Clock skew** can invalidate JWTs early.
- **Token revocation** is hard with stateless JWTs.
- **Replay attacks** require nonce or short TTLs.

---

## 🧱 Line-by-Line Code Breakdown (JWT example)

```ts
import { Hono } from "hono";
import { sign, verify } from "hono/jwt";

const app = new Hono();

const SECRET = "super-secret";

app.post("/login", async (c) => {
  const { email, password } = await c.req.json();

  // 1. Validate credentials (fake example)
  if (email !== "demo@site.com" || password !== "pass") {
    return c.json({ error: "Invalid credentials" }, 401);
  }

  // 2. Issue token
  const token = await sign({ sub: email, role: "user" }, SECRET);
  return c.json({ token });
});

app.get("/me", async (c) => {
  const token = c.req.header("Authorization")?.replace("Bearer ", "");
  if (!token) return c.json({ error: "Missing token" }, 401);

  const payload = await verify(token, SECRET);
  return c.json({ user: payload.sub, role: payload.role });
});
```

### Line-by-line explanation

- `sign()` creates a JWT with a payload and secret.
- `verify()` validates signature and decodes the payload.
- The `/me` route returns the authenticated user identity.

---

## 🛠 Active Engineering Exercise

1. Implement a login route with **hashed** passwords.
2. Add a refresh token endpoint.
3. Force access tokens to expire in 10 minutes.

---

## Debugging mindset 🧪

- If tokens fail, check **clock time** and **secret mismatch**.
- If users stay logged in too long, check **refresh logic**.
