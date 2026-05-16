# Module 16 — Performance Deep Dive

**Goal:** Learn how to diagnose and improve performance using measurement, caching, and smart architecture.

---

## Purpose & system intent (before code)

Performance is **user experience**. Every 100ms matters. The goal is to reduce wasted work and avoid unnecessary I/O.

---

## 🧠 Mental Model First: The Fast Food Line

Fast systems aren’t just “faster code”—they are **shorter lines**, **fewer steps**, and **pre‑made components** (caches).

---

## 🧭 Visual Explanation

```text
Client → Network → App → DB → App → Client
```

### Latency breakdown

| Layer        | Typical cost |
| ------------ | ------------ |
| Network      | 10–100ms     |
| JSON parsing | 1–5ms        |
| DB query     | 5–200ms      |
| External API | 50–500ms     |

---

## 🔍 Deep Technical Explanation

### Key levers

- **Reduce I/O** (cache, batch queries).
- **Reduce payload size** (compression, only needed fields).
- **Parallelize** independent tasks.

### Hidden complexity

- **N+1 queries** cause linear slowdowns.
- **Overcaching** serves stale data.
- **Premature optimization** hides real bottlenecks.

---

## 🧱 Line-by-Line Code Breakdown (simple cache)

```ts
const cache = new Map<string, { value: any; expires: number }>();

async function getCached(key: string, loader: () => Promise<any>) {
  const now = Date.now();
  const hit = cache.get(key);
  if (hit && hit.expires > now) return hit.value;

  const value = await loader();
  cache.set(key, { value, expires: now + 30_000 });
  return value;
}
```

### Execution flow

```text
Request → Cache check → DB (miss) → Cache set → Response
```

---

## 🛠 Active Engineering Exercise

1. Add a cache to a frequently read endpoint.
2. Measure response times before and after.
3. Add a cache invalidation path.

---

## Debugging mindset 🧪

- If performance is bad, **measure before changing**.
- If a change slows things down, revert and inspect the bottleneck.
