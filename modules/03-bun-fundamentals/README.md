# Module 03 — Bun Fundamentals: The All-in-One Engine

**Goal:** Understand exactly what Bun is, why it replaces Node/npm/ts-node/Jest concurrently, and how to harness its extreme performance and utility in a production context.

## Outcomes

By the end of this module you will be able to:

- Explain the distinction between a JS Engine (v8/JSC) and a JS Runtime (Node/Bun).
- Utilize the Bun CLI to install dependencies, run scripts, and execute tests without configuration overhead.
- Utilize native Bun APIs for massive performance gains (e.g., File I/O, SQLite, Server).
- Debug execution flow in a native TypeScript runtime without compilation steps.

---

## Purpose & system intent (before code)

Bun collapses _five tools into one runtime_. That changes how you build, test, and deploy. The purpose here is to understand **where Bun replaces Node, npm, ts-node, and Jest**, and where the trade-offs live so you can make production-grade decisions.

## 🧠 Mental Model First: The Swiss Army Knife vs. The Tool Shed

In the past (Node.js ecosystem), building a backend was like managing an entire shed of tools:

- You needed **Node.js** to run JavaScript.
- You needed **npm** or **yarn** to install packages.
- You needed **tsc** and **ts-node** to transpile and run TypeScript because Node didn't understand it.
- You needed **nodemon** to restart the server on file saves.
- You needed **Jest** to run tests.

**Bun is the ultimate Swiss Army Knife.**
It is written in Zig (a low-level, high-performance systems language) and uses JavaScriptCore (the engine inside Safari), rather than V8 (Chrome/Node).
It natively speaks TypeScript. There is no transpilation phase. You write `.ts`, you run `bun index.ts`, and it executes immediately.

---

## 🧭 Visual Explanation: Runtime Architectures

### The Old Node.js Flow (The Tool Shed)

```text
[Developer writes index.ts]
      |
      V (ts-node / tsc process spins up -> SLOW)
[Typescript Compiler parses, checks, and generates index.js]
      |
      V (node process starts -> OK)
[V8 Engine reads index.js, JIT compiles to bytecode, executes]
```

### The Bun Flow (The Swiss Army Knife)

```text
[Developer writes index.ts]
      |
      V (bun run index.ts starts -> INSTANT)
[Zig-based transpiler reads TS, strips types, passes to JSCore]
      |
      V
[JavaScriptCore JIT compiles and executes simultaneously]
```

### System Architecture

Bun isn't just running your code; it wraps around standard Web APIs.

- `fetch()` is built-in.
- `WebSocket` is built-in.
- System-level APIs (`Bun.file`, `Bun.serve`, `Bun.sql`) heavily bypass generic bridges for ultra-fast C/Zig syscalls.

### Bun CLI quick reference

| Task         | Command                      | Notes                 |
| ------------ | ---------------------------- | --------------------- |
| Run TS       | `bun run src/index.ts`       | No ts-node needed     |
| Install deps | `bun install`                | Creates `bun.lockb`   |
| Add package  | `bun add zod`                | Same as `npm install` |
| Tests        | `bun test`                   | Jest-like API         |
| Watch mode   | `bun run --hot src/index.ts` | Fast hot reload       |

---

## 🔍 Deep Technical Explanation

### 1. Package Management

`bun install` is orders of magnitude faster than `npm install` because of a global module cache and optimized OS-level file linking (hardlinks/symlinks). When you install `zod` in Project A, and then install `zod` in Project B, Bun doesn't download it again—it hardlinks to the same spot on your SSD.

### 2. Built-in SQLite

Bun includes a built-in, highly optimized SQLite driver (`bun:sqlite`). Because it's integrated natively into the runtime rather than being an N-API addon, crossing the JS/C++ boundary is incredibly fast.

### 3. Testing

Bun ships with `bun test`, a test runner fully compatible with Jest (`expect`, `describe`, `it`). Because it boots instantly, TDD (Test Driven Development) becomes completely frictionless.

### TypeScript behavior (critical)

- Bun **strips types at runtime**. Types don’t exist once code runs.
- If you rely on type-only safety without runtime validation, you can still crash.
- Prefer Zod for inputs, and keep public interfaces narrow.

### Hidden complexity

- **Node compatibility gaps**: Some Node-only APIs don’t exist or behave differently.
- **ESM vs CJS**: Bun prefers ESM; be explicit in imports.
- **Hot reload**: `--hot` restarts on file change, but long-lived connections may reset.

---

## 🧱 Line-by-Line Code Breakdown

Let's look at how utilizing Bun-specific APIs differs from standard Node APIs.

```typescript
// index.ts

// 1. NATIVE SERVER
// Instead of importing 'http' module, Bun has it globally.
// Hono actually wraps this exact API under the hood when doing `export default { fetch: app.fetch }`.
Bun.serve({
  port: 3000,
  fetch(req) {
    return new Response("Hello from native Bun!", { status: 200 });
  },
});

// 2. NATIVE FILE I/O
// Bun treats files almost as primitive types. No heavy fs.readFile streams needed for basics.
// Bun.file() is lazy; it does not read the disk until you call .text() or .json()
const file = Bun.file("./package.json");

async function readConfig() {
  // Parses JSON out of the file in C++ space, avoiding JS-side memory bloat
  const packageJson = await file.json();
  console.log(`Project name is: ${packageJson.name}`);
}
```

### Execution Flow & Secret Complexity

1. Command: `bun run --hot index.ts`
2. Bun initializes its runtime. The `--hot` flag tells Bun to attach file watchers to the dependency tree.
3. Bun encounters `Bun.file('./package.json')`. It does **not** read your hard drive. It merely creates a reference pointer.
4. When `file.json()` is invoked, Bun dispatches an asynchronous, non-blocking I/O call to the kernel.
5. While the OS reads the SSD, Bun can handle other HTTP requests.
6. When the file is read, Bun leverages low-level Zig code to parse the JSON string _before_ handing the JS Object back to your code. This is significantly faster than Node's `JSON.parse(fs.readFileSync())`.

---

## Debugging mindset 🧪

- If code “works in Node but not in Bun,” check **Node API compatibility** first.
- If tests are flaky, check **global state** (Bun runs tests fast and in parallel).

---

## 🛠 Active Engineering Exercise

**Your Task:**

1. Delete `node_modules` and `package-lock.json` if you have them.
2. Run `bun install`. Watch how it takes milliseconds to create a `bun.lockb` (a binary lockfile format unique to Bun).
3. Create a file called `math.test.ts`.
4. Write a simple test:

```typescript
import { expect, test } from "bun:test";

test("math works", () => {
  expect(2 + 2).toBe(4);
});
```

5. Run `bun test`. Feel the instant execution speed.
