# Hono + Bun Backend Engineering Course

You’re building production-grade backend instincts, not just “how to make requests work.” This course is structured like a real team mentorship: I’ll explain systems first, then we’ll build them, then I’ll make you defend your decisions. Expect tradeoffs, debugging drills, and architecture habits throughout.

## How this course works

- **Mental model first**: we start with intuition and analogies.
- **Visuals before syntax**: every lesson uses diagrams and flowcharts.
- **Deep explanation**: you’ll learn _why_ each tool exists, _when_ to use it, and _when not to_.
- **Guided → independent**: you’ll start guided, then ship features on your own with hints.
- **Production mindset**: we’ll discuss reliability, security, observability, and scale.

## Project arc

We’ll evolve a single backend from a tiny “hello API” into a production-ready service:

1. **PulseNotes API** — a clean, documented REST API.
2. **PulseNotes Pro** — auth, RBAC, caching, observability.
3. **PulseNotes Live** — realtime events, queues, background jobs.

Each module adds a real-world capability and refactors architecture to stay maintainable.

## Roadmap

| Module | Topic                       | Focus                                            | Milestone               | Folder                                                                                 |
| ------ | --------------------------- | ------------------------------------------------ | ----------------------- | -------------------------------------------------------------------------------------- |
| 00     | Backend & HTTP Fundamentals | Mental models, HTTP semantics, request lifecycle | Minimal Bun HTTP server | [modules/00-backend-http-fundamentals](modules/00-backend-http-fundamentals/README.md) |
| 01     | Backend Fundamentals        | Server responsibilities, layering, boundaries    | Service layer sketch    | [modules/01-backend-fundamentals](modules/01-backend-fundamentals/README.md)           |
| 02     | HTTP Fundamentals           | Methods, status codes, headers, caching          | API behavior spec       | [modules/02-http-fundamentals](modules/02-http-fundamentals/README.md)                 |
| 03     | Bun Fundamentals            | Runtime, bun run/install, bunx, env              | Bun project baseline    | [modules/03-bun-fundamentals](modules/03-bun-fundamentals/README.md)                   |
| 04     | Hono Fundamentals           | Hono app, context, handlers                      | First Hono API          | [modules/04-hono-fundamentals](modules/04-hono-fundamentals/README.md)                 |
| 05     | Routing                     | REST design, nested routers                      | Versioned routes        | [modules/05-routing](modules/05-routing/README.md)                                     |
| 06     | Request/Response Handling   | DTOs, parsing, content negotiation               | Robust I/O layer        | [modules/06-request-response-handling](modules/06-request-response-handling/README.md) |
| 07     | Middleware                  | Cross-cutting concerns                           | Logging + tracing       | [modules/07-middleware](modules/07-middleware/README.md)                               |
| 08     | Validation                  | Zod schemas, safe inputs                         | Validation pipeline     | [modules/08-validation](modules/08-validation/README.md)                               |
| 09     | Databases                   | PostgreSQL + Drizzle, migrations                 | Persistence layer       | [modules/09-databases](modules/09-databases/README.md)                                 |
| 10     | Authentication              | Sessions, JWT, OAuth basics                      | Authenticated API       | [modules/10-authentication](modules/10-authentication/README.md)                       |
| 11     | Authorization               | RBAC/ABAC, policy checks                         | Permissions system      | [modules/11-authorization](modules/11-authorization/README.md)                         |
| 12     | Error Handling              | Error taxonomy, problem+json                     | Consistent errors       | [modules/12-error-handling](modules/12-error-handling/README.md)                       |
| 13     | Testing                     | Unit, integration, contract                      | Bun test suite          | [modules/13-testing](modules/13-testing/README.md)                                     |
| 14     | Production Architecture     | Modules, boundaries, layering                    | Service modularization  | [modules/14-production-architecture](modules/14-production-architecture/README.md)     |
| 15     | Security                    | Threat modeling, hardening                       | Secure defaults         | [modules/15-security](modules/15-security/README.md)                                   |
| 16     | Performance                 | Latency budgeting, caching                       | Faster endpoints        | [modules/16-performance](modules/16-performance/README.md)                             |
| 17     | OpenAPI                     | Contract-first APIs                              | OpenAPI docs            | [modules/17-openapi](modules/17-openapi/README.md)                                     |
| 18     | Deployment                  | Docker, env, CI/CD                               | Production deploy       | [modules/18-deployment](modules/18-deployment/README.md)                               |
| 19     | Realtime APIs               | SSE/WebSockets                                   | Live updates            | [modules/19-realtime-apis](modules/19-realtime-apis/README.md)                         |
| 20     | Capstone Projects           | Design + build a full backend                    | Final portfolio system  | [modules/20-capstone-projects](modules/20-capstone-projects/README.md)                 |

## How to proceed

Open **Module 00** and follow the lesson. After each exercise, reply with your solution or questions. I’ll review like a senior on your team and push you deeper.

---

If you want a different project theme (e.g., e-commerce, analytics, or fintech), tell me now and we’ll swap the arc.
