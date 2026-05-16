# Module 18 — Deployment Deep Dive

**Goal:** Learn how to ship Hono + Bun apps reliably with proper environments, health checks, and rollback strategies.

---

## Purpose & system intent (before code)

Deployment is **not copying files**. It’s the process of safely replacing a running system with a new one without downtime or data loss.

---

## 🧠 Mental Model First: Airplane Maintenance

You don’t stop the entire airport to fix one plane. Deployments should be **gradual and safe**, with fallbacks.

---

## 🧭 Visual Explanation

```text
CI → Build → Deploy → Health Check → Traffic Switch
```

### Environments

| Environment | Purpose              |
| ----------- | -------------------- |
| Dev         | Fast iteration       |
| Staging     | Prod-like validation |
| Prod        | Real users           |

---

## 🔍 Deep Technical Explanation

### Health checks

- `/health` endpoint should validate DB + dependencies.
- Load balancers should only send traffic to healthy nodes.

### Hidden complexity

- **Database migrations** can break backward compatibility.
- **Cold starts** introduce latency (serverless).
- **Secrets** must never be baked into images.

---

## 🧱 Line-by-Line Code Breakdown (health endpoint)

```ts
app.get("/health", async (c) => {
  // Ideally check DB connection too
  return c.json({ ok: true, timestamp: Date.now() });
});
```

---

## 🛠 Active Engineering Exercise

1. Add a `/health` endpoint that checks DB connectivity.
2. Simulate a failed health check and verify traffic stops.
3. Create a rollback plan for failed deploys.

---

## Debugging mindset 🧪

- If deploy fails, check **logs** and **health checks** first.
- If latency spikes after deploy, compare **build artifacts** and **config**.
