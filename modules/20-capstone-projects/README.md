# Module 20 — Capstone Projects Deep Dive

**Goal:** Combine everything into real-world, production-ready systems. You’ll design, build, test, secure, and deploy a full backend.

---

## Purpose & system intent (before code)

Capstones are where theory becomes **engineering judgment**. The goal is not just to “finish,” but to prove you can **own a production system end-to-end**.

---

## 🧠 Mental Model First: The Startup Pitch

You’re not building a toy. You’re building a backend someone could **bet their product on**.

---

## 🧭 Visual Explanation

```text
Client → API → Service Layer → DB/Cache → Observability
```

### Capstone options

| Project        | Description                    |
| -------------- | ------------------------------ |
| Task Manager   | CRUD + auth + RBAC + search    |
| Realtime Chat  | WebSocket + auth + persistence |
| E‑commerce API | Orders + payments + inventory  |

---

## 🔍 Deep Technical Explanation

### Key requirements

- Layered architecture (controller → service → repository).
- Full validation + error handling.
- Auth + authorization.
- OpenAPI documentation.
- Deployment + health checks.

### Hidden complexity

- **Data migrations** when schema evolves.
- **Observability** for production debugging.
- **Scaling** under load.

---

## 🧱 Capstone Roadmap (Milestones)

1. **Design**: write API spec and DB schema.
2. **Build core**: CRUD + validation.
3. **Add auth**: JWT + roles.
4. **Observability**: logs, metrics, error tracking.
5. **Deploy**: staging + production.

---

## 🛠 Active Engineering Exercise

Pick one project and deliver:

- Complete API with tests.
- DB migrations.
- CI checks (lint + test).
- Deployment guide.

---

## Evaluation rubric

| Category    | Excellent                         | Needs work          |
| ----------- | --------------------------------- | ------------------- |
| Correctness | All endpoints behave as specified | Edge cases fail     |
| Security    | Auth + validation everywhere      | Missing checks      |
| Performance | Caching + indexing                | Slow queries        |
| Docs        | OpenAPI + README                  | Missing or outdated |

---

## Debugging mindset 🧪

- If issues appear only under load, run a **load test**.
- If prod differs from dev, compare **env vars** and **config**.
