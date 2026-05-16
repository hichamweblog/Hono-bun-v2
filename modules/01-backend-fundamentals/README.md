# Module 01 — Backend Fundamentals: The Engine Room of the Web

**Goal:** Establish an elite-level mental model of what a backend truly does, how responsibilities are separated in a production environment, and how to design clean, decoupled boundaries that scale gracefully from day 1 to day 1000.

## Outcomes

By the end of this module you will be able to:

- Formulate the core responsibilities of a backend service beyond just "sending JSON."
- Construct rigid, decoupled boundaries between Transport, Business Logic, and Data Access.
- Design service interfaces that represent unbreakable contracts with the frontend.
- Engineer systems with a clear distinction between "accidental complexity" and "essential complexity."

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

## 🧠 Mental Model First: The Airport Security Checkpoint

Imagine an international airport. The front doors are open to everyone (Frontend). Passengers (Requests) walk in carrying luggage (Payloads).
But before anyone boards a plane (Mutates Database State / Executes Business Logic), they must pass through a strict sequence of checks:

1. **Boarding Pass Check (Routing/Transport):** Are you even supposed to be here? Are you going to the right terminal?
2. **TSA Screening (Validation/Middleware):** Are you carrying anything dangerous? Does your bag fit the size requirements (Zod Validation)?
3. **Immigration/Customs (Authentication/Authorization):** Who are you? Do you have a visa to enter this specific area?
4. **The Gate Agent (Controller/Service Layer):** Finally applying the actual business rule—taking your ticket, assigning your seat, and loading you onto the plane.

A backend is a **paranoid, contract‑enforcing sequence of gates**. It processes chaotic, untrusted inputs from the outside world and transforms them into predictable, safe, and structured data execution. If an airport let people walk straight from the curb onto the plane (tightly coupled transport and business logic), chaos would ensue. Your backend is no different.

---

## 🧭 Visual Explanation: Architecture & Layers

### The Layered "Onion" Architecture

To keep the airport secure, we use layers. Each layer has **exactly one job** and only talks to the layer immediately below it.

```text
+-------------------------------------------------------------+
|                     EXTERNAL WORLD (HTTP/WSS)               |
+-------------------------------------------------------------+
                              || (JSON Payload / Request)
                              \/
+=============================================================+
| 1. TRANSPORT LAYER (Controllers / Routers)                  |
|    - Parses HTTP headers, query params, body.               |
|    - Does NOT know what a user is, only knows HTTP types.   |
+=============================================================+
                              || (Parsed, Raw Data)
                              \/
+=============================================================+
| 2. VALIDATION LAYER (Zod / DTOs)                            |
|    - Ensures the data shape matches the exact contract.     |
|    - Strips malicious or unexpected fields.                 |
+=============================================================+
                              || (Strongly Typed, Safe Data)
                              \/
+=============================================================+
| 3. BUSINESS LOGIC LAYER (Services / Use Cases)              |
|    - The 'Brain'. Applies domain rules (e.g., "User must    |
|      be 18+", "Check if email exists").                     |
|    - Does NOT know HTTP exists. Returns pure Data/Errors.   |
+=============================================================+
                              || (Domain Entities)
                              \/
+=============================================================+
| 4. DATA ACCESS LAYER (Repositories / ORM)                   |
|    - Executes SQL, talks to Redis, writes to S3.            |
|    - Does NOT know about business rules. Just stores data.  |
+=============================================================+
```

### Table: Layer Responsibilities

| Layer          | Good (Do This)                                  | Bad (Avoid This)                                      |
| -------------- | ----------------------------------------------- | ----------------------------------------------------- |
| **Transport**  | Extracting `req.body.id` and passing to Service | Writing `SELECT * FROM users` in the route            |
| **Validation** | Throwing 400 Bad Request if `age < 18`          | Checking if `age < 18` deep inside the database query |
| **Service**    | Calculating subscription discounts              | Setting `res.cookie('auth')` inside the Service       |
| **Repository** | `db.insert(user).values({...})`                 | Throwing a HTTP 404 Error if user not found           |

### Data shape flow (DTO → Domain → Persistence)

```text
Inbound JSON
  │ (unknown, untrusted)
  ▼
DTO (validated input)
  │ (safe, narrow)
  ▼
Domain Model (business rules)
  │ (rich behavior)
  ▼
Persistence Model (DB schema)
```

| Shape       | Example           | Purpose        | Hidden Complexity                      |
| ----------- | ----------------- | -------------- | -------------------------------------- |
| DTO         | `{ email, age }`  | Input contract | Missing fields, extra fields, coercion |
| Domain      | `User` entity     | Business rules | Invariants, lifecycle, transitions     |
| Persistence | `users` table row | Storage        | Migrations, indexes, defaults          |

---

## 🔍 Deep Technical Explanation

### 1. Separation of Concerns (SoC)

Without SoC, your code becomes a "Big Ball of Mud". If you write database queries inside your HTTP route handlers, you can never reuse that query for a Cron Job, a WebSocket message, or a CLI script.

### 2. The DTO (Data Transfer Object) Pattern

Data that enters your system is untrusted. It must be converted into a DTO—a rigidly defined TS interface or Zod schema—before it goes deeper. The hidden complexity here is that the shape of the data entering the API (e.g., password string) is rarely the same as the shape stored in the DB (hashed password).

### 3. Dependency Rule

Inner layers (Business, Data) MUST NEVER depend on outer layers (HTTP). The Business layer should never import `hono` or `express`. It should only take pure TypeScript interfaces as arguments.

### 4. Execution Flow (what really happens)

```text
HTTP Request
  │
  ▼
Controller → DTO parse → Service → Repository → DB
  │                                  ▲
  ▼                                  │
Response <────────────── Domain result/error
```

### 5. TypeScript behavior (critical)

- **DTOs protect runtime**, but **Types protect compile time**. You need both.
- Avoid exporting raw database rows. Use a mapped type or a presenter.
- Prefer `Result` or domain errors over throwing generic `Error` strings.

### Hidden complexity

- **Error translation**: Domain errors must map to HTTP errors cleanly.
- **Time-dependent rules**: “trial expires in 7 days” is business logic, not controller code.
- **Transactions**: cross-repo consistency is not trivial and should live in the service layer.

---

## 🧱 Line-by-Line Code Breakdown

Let's look at the difference between "Junior/Muddy" code and "Production-Grade Layered" code.

### ❌ The "Bad" Way (Tightly Coupled)

```typescript
// index.ts
import { Hono } from "hono";
import { db } from "./db"; // Direct DB access

const app = new Hono();

app.post("/users", async (c) => {
  // Transport, Validation, Business Logic, and Data all mixed!
  const body = await c.req.json();

  if (!body.email) return c.json({ error: "Email required" }, 400); // Validation

  const existingUser = await db.query(
    `SELECT * FROM users WHERE email = ?`,
    body.email,
  ); // DB Access
  if (existingUser) return c.json({ error: "Email taken" }, 400); // Business Logic

  await db.query(`INSERT INTO users (email) VALUES (?)`, body.email); // DB Access

  return c.json({ success: true }, 201); // Transport
});
```

**Why this fails at scale:**

1. Hard to test. You have to mock an entire HTTP server just to test user creation.
2. If we want to create a user via a terminal script, we have to copy-paste the whole logic.

---

### ✅ The "Elite" Way (Decoupled Layers)

Let's break this out intuitively.

#### 1. The Validation & Domain Contract (Zod)

We define our shape. This is the contract for the whole system.

```typescript
// schema.ts
import { z } from "zod";

// DTO: Defines exactly what the external world is allowed to send
export const CreateUserSchema = z.object({
  email: z.string().email("Must be a valid email"),
  age: z.number().min(18, "Must be an adult"),
});

export type CreateUserDTO = z.infer<typeof CreateUserSchema>;
```

- **Line-by-line explanation:**
  - `z.string().email()`: The Zod runtime engine will automatically regex match the string.
  - `z.infer`: We extract the pure TypeScript type from the runtime Zod object, so we don't have to duplicate our type definitions.

#### 2. The Service Layer (The Brain)

Notice how this file knows **nothing** about HTTP, Status codes, or Hono.

```typescript
// userService.ts
import type { CreateUserDTO } from "./schema";
import { db } from "./db"; // Fake repository

export class UserService {
  // Injecting dependencies or calling repositories
  static async createUser(data: CreateUserDTO) {
    // 1. Business logic check
    const exists = await db.findUserByEmail(data.email);
    if (exists) {
      // ⚠️ Notice: We throw a domain error, NOT an HTTP 400!
      throw new Error("UserAlreadyExistsError");
    }

    // 2. Data modification
    const newUser = await db.insertUser({
      email: data.email,
      age: data.age,
      createdAt: new Date(),
    });

    return newUser;
  }
}
```

#### 3. The Transport Layer (The Gate)

This layer handles Hono, parsers, and HTTP statuses.

```typescript
// userController.ts
import { Hono } from "hono";
import { CreateUserSchema } from "./schema";
import { UserService } from "./userService";

export const userRouter = new Hono();

userRouter.post("/", async (c) => {
  try {
    // 1. Validation (Extract & Parse)
    const rawData = await c.req.json();
    const parsedData = CreateUserSchema.parse(rawData); // Throws if invalid

    // 2. Pass to Service Layer
    const user = await UserService.createUser(parsedData);

    // 3. Return HTTP Response
    return c.json({ data: user }, 201);
  } catch (err: any) {
    // 4. Transform domain errors to HTTP errors
    if (err.name === "ZodError") {
      return c.json({ error: "Validation Failed", details: err.errors }, 400);
    }
    if (err.message === "UserAlreadyExistsError") {
      return c.json({ error: "Email already in use" }, 409);
    }
    return c.json({ error: "Internal Server Error" }, 500);
  }
});
```

_(Execution Flow)_
The Request arrives at `userRouter.post` → Context parses it to JSON → Zod verifies the shape (and fails early if bad) → The DTO is passed cleanly into `UserService.createUser` → The Service looks up the DB and inserts the record → The Service returns a real User object → The Router formats this into a JSON 201 response. If any step fails, the `catch` block intercepts it and translates it back into proper HTTP language (400, 409, 500).

---

## Debugging mindset 🧪

When something breaks, locate the layer first:

- **Transport bug**: wrong path/method, missing header → fix controller/router.
- **Validation bug**: bad payload shape → fix Zod schema.
- **Business bug**: rule wrong or missing → fix service.
- **Data bug**: query, index, transaction → fix repository.

**Rule:** If you can't point to the layer, you can’t fix it reliably.

---

## 🛠 Active Engineering Exercise

**Your Task:**
Identify the layers in your current projects. Have you mixed Business Logic with Transport? Start thinking about how you could separate them into `Controller` -> `Service` -> `Repository`.

Next, we will explore exactly how HTTP works under the hood built on top of this model.
