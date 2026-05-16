# Module 13 — Testing Deep Dive

**Goal:** Build a reliable testing strategy: fast unit tests, meaningful integration tests, and safe production deploys.

---

## Purpose & system intent (before code)

Testing is your **safety net**. The stronger the net, the faster you can move without fear.

---

## 🧠 Mental Model First: The Crash Test

You don’t test to prove it works once—you test to prove it **keeps working** when things change.

---

## 🧭 Visual Explanation

```text
Unit Tests (fast) → Integration Tests (real DB) → E2E (full system)
```

### Test pyramid

| Layer       | Speed  | Value             |
| ----------- | ------ | ----------------- |
| Unit        | Fast   | Logic correctness |
| Integration | Medium | DB + services     |
| E2E         | Slow   | Full workflow     |

---

## 🔍 Deep Technical Explanation

### Hidden complexity

- **Flaky tests** are worse than no tests.
- **Global state** breaks test isolation.
- **Time-dependent logic** should use fake timers.

### TypeScript behavior

- Tests should compile with the same strictness as production.
- Prefer typed fixtures to avoid invalid data.

---

## 🧱 Line-by-Line Code Breakdown (Bun + Hono)

```ts
import { describe, expect, it } from "bun:test";
import { Hono } from "hono";

const app = new Hono();
app.get("/health", (c) => c.json({ ok: true }));

describe("/health", () => {
  it("returns ok", async () => {
    const res = await app.request("/health");
    const body = await res.json();
    expect(body.ok).toBe(true);
  });
});
```

### Execution flow

```text
Test → app.request → handler → response → assertions
```

---

## 🛠 Active Engineering Exercise

1. Write a unit test for a pure service function.
2. Write an integration test with a test database.
3. Add a failing test on purpose and fix it.

---

## Debugging mindset 🧪

- If tests fail intermittently, check **shared state** or **time-based logic**.
- If tests are slow, move heavy setup to `beforeAll`.
