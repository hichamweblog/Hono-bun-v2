# Module 15 — Security Deep Dive

**Goal:** Build a security-first mindset: protect data, defend against common attacks, and reduce your system’s attack surface.

---

## Purpose & system intent (before code)

Security is **risk management**. You cannot make a system perfectly secure, but you can make it **expensive to attack** and **safe to operate**.

---

## 🧠 Mental Model First: The Castle Walls

Security is layered: outer walls (network), inner gates (auth), and guards (validation). If one fails, the next layer still protects you.

---

## 🧭 Visual Explanation

```text
Request → WAF → Auth → Validation → Business Logic → DB
```

### OWASP Top Risks (shortlist)

| Risk               | Example             | Defense                     |
| ------------------ | ------------------- | --------------------------- |
| Injection          | SQL injection       | Parameterized queries       |
| Broken Auth        | Weak tokens         | Strong JWT/Session strategy |
| Sensitive Data     | Plaintext passwords | Hash + TLS                  |
| Security Misconfig | Open CORS           | Lock down origins           |

---

## 🔍 Deep Technical Explanation

### Key defenses

- **Validate input** with Zod.
- **Hash passwords** (argon2/bcrypt).
- **Rate limit** login endpoints.
- **Use HTTPS** everywhere.
- **Set security headers** (`Content-Security-Policy`, `X-Frame-Options`).

### Hidden complexity

- **CORS** mistakes expose APIs to untrusted origins.
- **CSRF** is real when cookies are used.
- **Secret leakage** in logs or error messages.

---

## 🧱 Line-by-Line Code Breakdown (Rate limiting)

```ts
import { Hono } from "hono";

const app = new Hono();

const hits = new Map<string, { count: number; reset: number }>();

app.use("/login", async (c, next) => {
  const ip = c.req.header("x-forwarded-for") || "local";
  const now = Date.now();

  const entry = hits.get(ip) ?? { count: 0, reset: now + 60_000 };
  if (now > entry.reset) {
    entry.count = 0;
    entry.reset = now + 60_000;
  }

  entry.count++;
  hits.set(ip, entry);

  if (entry.count > 10) {
    return c.json({ error: "Too many attempts" }, 429);
  }

  await next();
});
```

---

## 🛠 Active Engineering Exercise

1. Add rate limiting to `/login`.
2. Add `Content-Security-Policy` header.
3. Ensure password hashes are never logged.

---

## Debugging mindset 🧪

- If auth fails sporadically, check **clock drift** and **token expiry**.
- If CORS fails, check **origin and allowed methods**.
