# Module 19 — Realtime APIs Deep Dive

**Goal:** Understand real‑time patterns (WebSockets, SSE, long polling) and how to build them safely.

---

## Purpose & system intent (before code)

Realtime systems are about **continuous state synchronization**. They introduce new failure modes: dropped connections, message ordering, and backpressure.

---

## 🧠 Mental Model First: Radio Broadcast vs Phone Call

- **SSE** = radio broadcast (server talks, clients listen).
- **WebSocket** = phone call (two‑way conversation).

---

## 🧭 Visual Explanation

```text
Client ⇄ WebSocket (bidirectional)
Client ← SSE (server push)
Client ←→ Long Poll (repeated requests)
```

### Choose the right tool

| Pattern   | Use case             | Tradeoffs             |
| --------- | -------------------- | --------------------- |
| SSE       | Notifications, feeds | Server → Client only  |
| WebSocket | Chat, multiplayer    | More infra complexity |
| Long Poll | Legacy               | Higher latency        |

---

## 🔍 Deep Technical Explanation

### Hidden complexity

- **Reconnect logic**: clients must recover from disconnects.
- **Ordering**: messages can arrive out of order.
- **Backpressure**: slow clients can block the server.

---

## 🧱 Line-by-Line Code Breakdown (SSE example)

```ts
app.get("/events", (c) => {
  const stream = new ReadableStream({
    start(controller) {
      const encoder = new TextEncoder();
      controller.enqueue(encoder.encode("data: hello\n\n"));

      const interval = setInterval(() => {
        controller.enqueue(encoder.encode(`data: ${Date.now()}\n\n`));
      }, 1000);

      return () => clearInterval(interval);
    },
  });

  return new Response(stream, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      Connection: "keep-alive",
    },
  });
});
```

---

## 🛠 Active Engineering Exercise

1. Implement SSE notifications for `/events`.
2. Add a client reconnect strategy.
3. Add a heartbeat message every 10 seconds.

---

## Debugging mindset 🧪

- If clients disconnect, check **timeouts and keep‑alive headers**.
- If messages are missing, check **buffering and backpressure**.
