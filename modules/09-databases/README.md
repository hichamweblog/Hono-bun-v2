# Module 09 — Databases Deep Dive

**Goal:** Build a practical, production-safe mental model of databases: schema design, queries, transactions, and performance.

---

## Purpose & system intent (before code)

Databases are **the source of truth**. If you misuse them, everything else becomes unreliable. This module teaches you how to model data, query it safely, and keep integrity under concurrency.

---

## 🧠 Mental Model First: The Bank Ledger

Databases are like bank ledgers: every write must be **consistent**, every read must be **accurate**, and every update must be **atomic**.

---

## 🧭 Visual Explanation

```text
Service Layer → Repository → SQL → Database
```

### ACID in one table

| Property    | Meaning                    | Why it matters           |
| ----------- | -------------------------- | ------------------------ |
| Atomicity   | All or nothing             | No half-written records  |
| Consistency | Rules always true          | Prevents invalid states  |
| Isolation   | Transactions don't collide | Prevents race conditions |
| Durability  | Writes persist             | Survive crashes          |

---

## 🔍 Deep Technical Explanation

### Indexes (performance lever)

- Indexes speed reads but slow writes.
- Use indexes for **frequent filters** and **joins**.

### Transactions (integrity lever)

- If a workflow touches multiple tables, wrap it in a transaction.
- If any step fails, rollback everything.

### TypeScript behavior

- ORMs generate types, but validate input anyway.
- Don’t expose raw DB rows directly to clients.

### Hidden complexity

- **N+1 queries**: repeated queries inside loops kill performance.
- **Connection pooling**: too many connections can crash the DB.
- **Migrations**: schema changes must be planned.

---

## 🧱 Line-by-Line Code Breakdown (Drizzle example)

```ts
import { drizzle } from "drizzle-orm/bun-sql";
import { sql } from "drizzle-orm";
import { Database } from "bun:sqlite";

const sqlite = new Database("./dev.db");
const db = drizzle(sqlite);

// 1) Create table (example)
await db.run(sql`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    created_at TEXT NOT NULL
  );
`);

// 2) Insert
await db.run(sql`
  INSERT INTO users (email, created_at)
  VALUES ('alice@example.com', datetime('now'));
`);

// 3) Select
const rows = await db.all(sql`SELECT * FROM users`);
```

### Execution flow

```text
Service → Repository → SQL → DB → Rows → Mapper → Response
```

---

## 🛠 Active Engineering Exercise

1. Add a `users` table with `email` and `created_at`.
2. Write a query to fetch users created in the last 7 days.
3. Add an index on `created_at` and compare query speed.

---

## Debugging mindset 🧪

- If a query is slow, inspect **indexes** first.
- If results are wrong, confirm **transaction boundaries**.
