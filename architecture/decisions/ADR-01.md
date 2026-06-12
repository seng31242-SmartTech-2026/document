# ADR-01 — Architectural Pattern Selection

| Field             | Detail                                                      |
|-------------------|-------------------------------------------------------------|
| **ADR Number**    | ADR-01                                                      |
| **Title**         | Layered MVC Architecture with REST API for SmartTech        |
| **Date**          | 2026-05-10                                                  |
| **Status**        | Accepted                                                    |
| **Deciders**      | Lead Architect (Piranavi Sasikaran), Code Crafters Team     |
| **Milestone**     | Sprint 3                                                    |
| **Related Issues**| [#12](https://github.com/seng31242-SmartTech-2026/document/issues/12) · [#14](https://github.com/seng31242-SmartTech-2026/document/issues/14) · [#15](https://github.com/seng31242-SmartTech-2026/document/issues/15) |
| **References**    | Course Guideline §7.2 — Architectural Design Standards; §7.2.1 — ADR Template |

---

## 1. Context

SmartTech is a multi-sided SaaS marketplace for mobile phones, laptops, and accessories, connecting Buyers, Sellers, and Administrators through a single web-based platform. At the outset of Sprint 3 the team must formally select and record the overarching architectural pattern that will govern every subsequent design and implementation decision.

The system will be developed by a four-person team within a fixed academic timeline (SENG 31242, 2024/2025). The initial open-source release must be deployable on cloud free-tier infrastructure, maintainable by the team with no dedicated DevOps resource, and extensible to higher traffic in later versions without a complete rewrite.

The key constraints driving this decision are:

- **Team skill set** — all four members have working proficiency in React.js, Node.js/Express, and relational databases; none have production microservices operations experience.
- **Budget** — deployment must be viable on free-tier cloud services (DigitalOcean App Platform, Supabase, AWS S3 free tier).
- **Deadline** — full working system must be delivered within the academic semester.
- **Supervisor review** — the architecture must be formally documented and reviewable against IEEE Std 1016-2009 and Course Guideline §7.2.

---

## 2. Decision

> **SmartTech will use a Layered (N-Tier) MVC architecture with a RESTful API.**

Concretely, the system is structured as three tiers:

| Tier | Technology | Role |
|------|-----------|------|
| **Presentation** | React.js 18 SPA (Tailwind CSS) | Renders all UI; communicates with the API via HTTPS REST |
| **Business Logic** | Node.js 20 LTS / Express 4 | MRCS pattern: Routes → Controllers → Services → Repositories |
| **Data** | PostgreSQL 15 + AWS S3 | Relational persistence; binary object storage for images |

Within the business logic tier, the **Model-Route-Controller-Service (MRCS)** pattern is applied — a REST-adapted variant of MVC that isolates routing, HTTP handling, business logic, and data access into four explicit layers.

All API contracts follow REST principles: stateless, resource-oriented URLs, standard HTTP verbs, and JSON payloads. Authentication uses JWT Bearer tokens (stateless, horizontally scalable).

---

## 3. Rationale

The decision is justified by direct traceability to Non-Functional Requirements from SRS Chapter 5:

### 3.1 NFR-Driven Justification

| NFR (SRS Ch. 5) | Requirement | How Layered MVC / REST Satisfies It |
|-----------------|-------------|-------------------------------------|
| **NFR-22 — Horizontal Scalability** | The system must support horizontal scaling of the API server without shared in-memory state. | The Express API is fully stateless: no server-side sessions, no local-disk persistence. JWT tokens carry all auth state. Multiple identical API instances can run behind a load balancer with zero code changes. |
| **NFR-25 — API Documentation** | All API endpoints must be documented via OpenAPI 3.0. | A REST API has a well-defined contract surface. `swagger-jsdoc` annotations on Express routes auto-generate the OpenAPI spec. The same workflow is not practical for GraphQL subscriptions or event-driven architectures without additional tooling. |
| **NFR-26 — Code Style Enforcement** | The codebase must pass ESLint Airbnb rules on every commit. | The MRCS pattern's explicit file structure (routes/, controllers/, services/, repositories/) maps directly to ESLint rule scopes and Husky pre-commit hooks. Microservices spread this across multiple repositories, complicating uniform lint enforcement. |
| **NFR-28 — 70 % Unit Test Coverage** | Service-layer unit test coverage ≥ 70 % enforced in CI. | The service layer in MRCS is a plain JavaScript class with no HTTP dependency; Jest can test it in isolation with mocked repositories. This is structurally harder to achieve in a flat single-file MVC or in a serverless function where business logic is co-located with HTTP handlers. |
| **NFR-01 — Page Load ≤ 3 s** | Product listing page must load within 3 seconds under 200 concurrent users on 4G. | The React SPA is served as a static CDN artefact (Netlify / Vercel), completely decoupled from the API server. Code-splitting (React.lazy + Suspense) and server-side pagination (20 items/page) keep initial payloads small. Serverless cold-start latency (NFR-01 conflict) and microservices inter-service latency were both evaluated and found to add unacceptable overhead. |
| **NFR-02 — Search ≤ 2 s** | Search results must be returned within 2 seconds for up to 10,000 listings. | A single PostgreSQL instance with a GIN full-text index on the product_listings table satisfies this requirement. Microservices would require a separate Search Service with its own data store synchronisation, adding latency and operational complexity unnecessary at this scale. |

### 3.2 Additional Rationale

- **Team velocity.** All four engineers are proficient in the chosen stack. Using a well-understood pattern eliminates the learning-curve tax on a fixed-duration project.
- **Operational simplicity.** A single deployable API artefact (Docker container on DigitalOcean App Platform) requires no service mesh, no container orchestration (Kubernetes), and no distributed tracing infrastructure.
- **Modular extraction path.** The MRCS service boundaries are designed so that individual services (e.g., `PaymentService`, `NotificationService`) can be extracted into standalone microservices in version 2.0 without rewriting business logic — satisfying the scalability intent of NFR-22 and NFR-23 without premature complexity.

---

## 4. Alternatives Considered

### 4.1 Alternative 1 — Microservices Architecture

| Dimension | Detail |
|-----------|--------|
| **Description** | Decompose the system into independently deployable services: `UserService`, `ProductService`, `OrderService`, `PaymentService`, `NotificationService`, each with its own database and exposed via an API Gateway. |
| **Pros** | Independent deployability and scalability per service; fault isolation; enables polyglot persistence (e.g., Redis for sessions, Elasticsearch for product search); aligns with industry best practice for high-scale systems. |
| **Cons** | Requires container orchestration (Kubernetes or Docker Swarm); distributed tracing and centralised logging become mandatory infrastructure; inter-service network latency adds to response times (conflicts with NFR-01, NFR-02); each service needs its own CI/CD pipeline; dramatically increases operational surface for a four-person team with no DevOps resource; free-tier budget insufficient for 5+ separate service deployments. |
| **Reason Rejected** | The operational overhead is disproportionate to the team size and academic timeline. NFR-01 (3 s page load) and NFR-02 (2 s search) are at risk from inter-service latency without a service mesh. The layered monolith (this decision) provides a clear extraction path to microservices in v2.0 once operational maturity is reached, without sacrificing current delivery velocity. |

### 4.2 Alternative 2 — Serverless / Backend-as-a-Service (BaaS)

| Dimension | Detail |
|-----------|--------|
| **Description** | Replace the Express API server with serverless functions (AWS Lambda + API Gateway or Firebase Cloud Functions). Use a BaaS platform (Firebase / Supabase) for authentication, real-time database, and file storage, eliminating most backend code. |
| **Pros** | Zero server management; automatic scaling to millions of requests; pay-per-invocation cost model suits low-traffic early stages; Firebase Auth eliminates custom JWT implementation; generous free tiers on Firebase / Supabase. |
| **Cons** | Cold-start latency (100–3000 ms per cold invocation) directly conflicts with NFR-01 (page load ≤ 3 s); complex business logic (multi-step order creation, payment webhook verification, stock reservation) is awkward to express as stateless functions with strict execution time limits; vendor lock-in to Firebase/AWS makes NFR-26 (code style) and NFR-28 (unit test coverage) harder to enforce uniformly; PayHere webhook verification requires reliable, always-warm HTTP endpoint behaviour. |
| **Reason Rejected** | Cold-start latency is an unacceptable risk for NFR-01. The order placement and payment webhook flows (UC-08, UC-09) require transactional, multi-step server-side logic that exceeds the practical scope of a single Lambda invocation. Vendor lock-in reduces the portability required for an open-source academic release. Unit testing serverless handlers to meet NFR-28's 70 % coverage threshold requires additional emulation tooling (AWS SAM, Firebase Emulator Suite) that adds friction without architectural benefit at this scale. |

### 4.3 Alternative 3 — Event-Driven / CQRS Architecture

| Dimension | Detail |
|-----------|--------|
| **Description** | Separate read and write models (CQRS) with an asynchronous message broker (Apache Kafka or RabbitMQ) coordinating eventual consistency between services. |
| **Pros** | Extremely high write throughput; audit log built into the event stream; read models can be optimised independently. |
| **Cons** | Requires running and operating a message broker; eventual consistency complicates the order-payment-stock flow (requires saga pattern); debugging distributed transactions demands specialised tooling; far exceeds team expertise and project timeline. |
| **Reason Rejected** | Complexity is unjustifiable at the current scale (NFR-03 targets 500 concurrent users). An internal Node.js `EventEmitter` is used within the monolith for the notification flow — preserving the architectural intent of decoupling notifications without introducing a full broker. |

---

## 5. Consequences

### 5.1 Positive Consequences

- Single deployable unit simplifies CI/CD: one GitHub Actions pipeline, one Docker image, one DigitalOcean App.
- Clear MRCS layer boundaries enable independent unit testing of service classes (Jest mocks, no HTTP transport required) — directly enabling NFR-28.
- Stateless API design (JWT, no server sessions) satisfies NFR-22 out of the box; adding a second API instance behind a load balancer requires zero code changes.
- OpenAPI spec auto-generated from Express JSDoc annotations satisfies NFR-25 with minimal overhead.
- Team can begin feature implementation immediately using familiar patterns and tooling.

### 5.2 Negative Consequences / Trade-offs

- **Monolith deployment coupling.** A bug in any layer (e.g., Admin Service) requires redeployment of the entire API. Mitigation: feature flags and blue/green deployment on DigitalOcean App Platform.
- **Database as single point of coupling.** All services share one PostgreSQL instance. Mitigation: schema-level separation by table prefix; connection pooling via `pg-pool`; read replicas available in Supabase paid tier if needed in v2.0.
- **Background job coordination.** The 30-minute order auto-cancel cron job (`node-cron`) is embedded in the API process. With multiple API instances, the job would fire multiple times. Mitigation: database-level idempotency check (`IF status = 'PENDING_PAYMENT'`) ensures duplicate runs are safe; a distributed job queue (BullMQ) is the recommended v2.0 upgrade path.

---

## 6. Compliance Checklist

| Acceptance Criterion | Status | Evidence |
|----------------------|--------|----------|
| AC1 — ADR-01 follows §7.2.1 template exactly | ✅ Met | This document; sections map to template fields |
| AC2 — Alternatives table covers Microservices and Serverless with Pros, Cons, Reason Rejected | ✅ Met | Section 4.1 (Microservices), 4.2 (Serverless/BaaS), 4.3 (CQRS) |
| AC3 — Rationale references ≥ 3 NFRs from SRS Ch. 5 | ✅ Met | Section 3.1: NFR-22, NFR-25, NFR-26, NFR-28, NFR-01, NFR-02 (6 NFRs) |
| AC4 — Component/package diagram in UML 2.x (draw.io) | ⏳ In Progress | `documents/diagrams/component-diagram.drawio` / `.svg` — see Issue #15 |
| AC5 — Deployment diagram showing 3 tiers | ⏳ In Progress | `documents/diagrams/deployment-diagram.drawio` / `.svg` — see Issue #15 |
| AC6 — ADR-01 committed to `documents/architecture/decisions/ADR-01.md` | ⏳ In Progress | This file; commit pending PR review |
| AC7 — Diagram `.drawio` source and SVG exports committed | ⏳ In Progress | Blocked on AC4/AC5 completion |

---

## 7. File Locations

```
documents/
├── architecture/
│   └── decisions/
│       └── ADR-01.md                          ← this file
└── diagrams/
    ├── component-diagram.drawio
    ├── deployment-diagram.drawio
    └── exports/
        ├── component-diagram.svg
        └── deployment-diagram.svg
```

---

## 8. Review Sign-off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Author / Lead Architect | Piranavi Sasikaran (SE/2022/054) | 2026-05-10 | |
| Team Lead Review | | | |
| Supervisor Review | | | |

---

*Document conforms to: IEEE Std 1016-2009 · Course Guideline §7.2.1 · SmartTech SDS v1.0*
