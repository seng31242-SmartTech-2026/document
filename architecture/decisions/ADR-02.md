# ADR-02 — Technology Stack Selection (MERN + Tailwind CSS + REST API)

| Field             | Detail                                                                 |
|-------------------|------------------------------------------------------------------------|
| **ADR Number**    | ADR-02                                                                 |
| **Title**         | MERN Stack + Tailwind CSS + REST API for SmartTech                     |
| **Date**          | 2026-05-10                                                             |
| **Status**        | Accepted                                                               |
| **Deciders**      | Lead Architect (Piranavi Sasikaran), Code Crafters Team                |
| **Milestone**     | Sprint 3                                                               |
| **Supersedes**    | —                                                                      |
| **Related ADRs**  | [ADR-01](./ADR-01.md) — Architectural Pattern Selection                |
| **Related Issues**| [#13](https://github.com/seng31242-SmartTech-2026/document/issues/13) · [#14](https://github.com/seng31242-SmartTech-2026/document/issues/14) · [#16](https://github.com/seng31242-SmartTech-2026/document/issues/16) |
| **References**    | Course Guideline §7.2.1 — ADR Template; §7.1.1 — SDS Chapter 3        |

---

## 1. Context

ADR-01 established that SmartTech will be built on a Layered MVC architecture with a RESTful API. This ADR records the specific technology selection for each layer of that architecture and justifies each choice against at least one realistic alternative, with traceability to Non-Functional Requirements (NFRs) from SRS Chapter 5.

The six technology decisions covered here are:

1. **MongoDB** — primary data store
2. **Express.js** — HTTP server / API framework
3. **React.js** — frontend SPA framework
4. **Node.js** — server-side JavaScript runtime
5. **Tailwind CSS** — utility-first CSS framework
6. **REST API** — API architectural style

These six choices together constitute the **MERN + Tailwind + REST** stack on which the entire SmartTech platform is built.

The same constraints identified in ADR-01 apply: four-person team, fixed academic semester, free-tier cloud budget, and full-stack JavaScript proficiency across all team members.

---

## 2. Decision

> **SmartTech will be built on MongoDB, Express.js, React.js, Node.js, Tailwind CSS, and a RESTful API architecture.**

| Layer | Technology | Version |
|-------|-----------|---------|
| Frontend framework | React.js | 18.x |
| Frontend styling | Tailwind CSS | 3.x |
| API architecture | REST (JSON over HTTPS) | — |
| Backend runtime | Node.js | 20 LTS |
| Backend framework | Express.js | 4.x |
| Primary database | MongoDB | 7.x (Atlas free tier) |

---

## 3. Individual Technology Justifications

### 3.1 MongoDB

**Decision:** Use MongoDB 7.x (hosted on MongoDB Atlas free tier) as the primary data store.

SmartTech's product catalogue presents a core data modelling challenge: mobile phones, laptops, and accessories each carry fundamentally different specification sets. A phone listing requires fields such as `batteryCapacity`, `simSlots`, and `displayResolution`; a laptop listing requires `ramGB`, `storageType`, and `gpuModel`; an accessory may require neither. In a traditional relational schema this forces either a sparse, nullable `specifications` table or a costly Entity-Attribute-Value (EAV) pattern — both of which degrade query performance and developer ergonomics.

MongoDB's document model allows each `ProductListing` document to embed a `specifications` sub-document whose shape is defined by the category, with no schema migration required when a new product type is introduced. This directly supports **NFR-23** (the system must support new product categories without a code deployment) and **NFR-24** (the storage layer must scale to 1 M+ product images and documents). Atlas Vector Search and Atlas Search (built on Apache Lucene) also provide a path to full-text product search that satisfies **NFR-02** (search results within 2 seconds for 10,000+ listings) without a separate Elasticsearch cluster.

**AC3 NFR references:** NFR-02 (search latency), NFR-23 (extensible categories), NFR-24 (storage scalability).

---

### 3.2 Express.js

**Decision:** Use Express.js 4.x as the HTTP server framework on top of Node.js.

Express is the de-facto standard HTTP framework for Node.js and the one with which all four team members have direct project experience. Its middleware pipeline maps cleanly onto the MRCS layering decided in ADR-01: route definitions are thin entry points that delegate immediately to controller functions, which in turn call service-layer classes. Authentication middleware (`verifyToken`), validation middleware (`express-validator`), rate-limiting middleware (`express-rate-limit`), and security-header middleware (`helmet`) can all be composed declaratively in route files without polluting business logic.

Express's minimal surface area means that `swagger-jsdoc` JSDoc annotations on route handlers auto-generate the OpenAPI 3.0 spec required by **NFR-25** (all endpoints must be documented). The framework imposes no opinions on project structure, making the ESLint Airbnb rule-set and Husky pre-commit hooks required by **NFR-26** straightforward to enforce uniformly.

**AC3 NFR references:** NFR-25 (OpenAPI documentation), NFR-26 (ESLint enforcement).

---

### 3.3 React.js

**Decision:** Use React.js 18.x with React Router v6 and Axios as the frontend SPA framework.

React's component model enables the SmartTech UI to be decomposed into independently testable, reusable units — a `ProductCard` used in both search results and the seller dashboard, a `StatusBadge` shared across order and listing tables. React Router v6 provides client-side navigation without full-page reloads, keeping perceived navigation latency well below the 3-second threshold of **NFR-01**. React 18's concurrent rendering features (Suspense, `React.lazy` code-splitting) reduce the initial JS bundle sent to the browser, directly addressing the page-load constraint under 200 concurrent users on 4G.

React's ecosystem breadth is also a risk-mitigation factor: every third-party integration required by SmartTech (PayHere redirect, SendGrid email trigger, AWS S3 pre-signed upload, Recharts analytics dashboard) has a mature React-compatible library or hook pattern. The community size means that team members can resolve implementation blockers quickly through documentation and community support, protecting **NFR-18** (system availability) by reducing the risk of blocked development.

**AC3 NFR references:** NFR-01 (page load ≤ 3 s), NFR-18 (availability / reduced development risk).

---

### 3.4 Node.js

**Decision:** Use Node.js 20 LTS as the server-side JavaScript runtime.

Node.js's event-driven, non-blocking I/O model is architecturally well-suited to SmartTech's workload profile: marketplace API calls are predominantly I/O-bound (MongoDB reads/writes, S3 URL generation, PayHere webhook handling, SendGrid email dispatch) rather than CPU-bound. A single Node.js process handles thousands of concurrent I/O operations without spawning a thread per request, enabling the platform to meet **NFR-03** (500 concurrent users without degradation exceeding 5 seconds) on free-tier infrastructure that would be insufficient for a thread-per-request server.

Using JavaScript on both frontend (React) and backend (Node.js/Express) eliminates the context-switching overhead of a polyglot stack. Team members can contribute to either tier without re-learning language idioms, protecting the sprint delivery schedule and indirectly supporting **NFR-28** (test coverage) by keeping the total codebase surface area manageable within the team's capacity.

**AC3 NFR references:** NFR-03 (concurrent user capacity), NFR-28 (test coverage maintainability).

---

### 3.5 Tailwind CSS

**Decision:** Use Tailwind CSS 3.x (JIT mode) as the utility-first styling framework.

SmartTech must be fully responsive from a 360 px mobile viewport to a 2560 px desktop display (**NFR-13**) and must meet WCAG 2.1 Level AA colour contrast requirements (**NFR-14**). Tailwind's utility-first approach provides the `sm:`, `md:`, `lg:`, and `xl:` responsive breakpoint prefixes directly in JSX markup, enabling mobile-first layouts without switching between HTML and a separate stylesheet. The JIT (Just-In-Time) compiler purges every unused utility class at build time, producing a production CSS bundle typically under 10 KB — contributing to the page-load budget of **NFR-01**.

Tailwind's design-token configuration (`tailwind.config.js`) provides a single source of truth for the SmartTech colour palette, font scale, and spacing scale. Accessibility-critical decisions — such as ensuring the primary action colour `#1A1A1A` on white achieves a ≥ 7:1 contrast ratio — are enforced at the token level and propagate automatically to every component. This is significantly more reliable than per-component style overrides in a traditional CSS framework.

**AC3 NFR references:** NFR-01 (CSS bundle size → page load), NFR-13 (mobile responsiveness), NFR-14 (WCAG 2.1 AA).

---

### 3.6 REST API Architecture

**Decision:** Expose all backend capabilities as a RESTful JSON API over HTTPS.

REST's resource-oriented, stateless request model is the most natural fit for the MRCS backend architecture decided in ADR-01. Each Express route maps directly to a resource (`/products`, `/orders`, `/cart`) with standard HTTP verbs (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`), producing an API surface that is self-documenting and familiar to every team member. Stateless requests — with all authentication state carried in a JWT Bearer token — are the architectural pre-condition for horizontal scaling required by **NFR-22**.

REST's broad tooling support enables the OpenAPI 3.0 documentation workflow (swagger-jsdoc → swagger-ui-express) mandated by **NFR-25** with minimal overhead. Postman collections, browser DevTools network inspection, and `curl` all work natively with a REST API, accelerating integration testing and debugging and reducing the effort required to achieve the 70 % test coverage threshold of **NFR-28**. All three external integrations — PayHere (payment webhook), SendGrid (email API), and AWS S3 (pre-signed URL generation) — expose REST interfaces, making the architectural style consistent end-to-end.

**AC3 NFR references:** NFR-22 (stateless horizontal scalability), NFR-25 (OpenAPI documentation), NFR-28 (testability).

---

## 4. Alternatives Considered

### 4.1 MongoDB vs PostgreSQL

| Dimension | MongoDB 7.x | PostgreSQL 15.x |
|-----------|-------------|-----------------|
| **Data model** | Document (BSON); flexible schema per document | Relational; strict schema with migrations |
| **Product specs** | Native nested `specifications` sub-document per category; no schema migration for new product types | Requires sparse nullable columns, JSONB column, or EAV table; JSONB is flexible but loses type enforcement |
| **Full-text search** | Atlas Search (Lucene-based) built-in; no separate cluster | GIN index on `tsvector`; capable but less feature-rich for faceted search |
| **ACID transactions** | Multi-document ACID since v4.0; supported but less ergonomic | First-class multi-table ACID; superior for financial/order data |
| **Team familiarity** | All four members — direct project experience | Two of four members — limited to coursework |
| **Free tier** | MongoDB Atlas M0 (512 MB storage, shared cluster) | Supabase / Render free tier (500 MB, shared instance) |
| **NFR alignment** | NFR-23 (extensible categories without deployment) ✅ | NFR-23 requires migration for new category fields ⚠️ |
| **Reason Rejected (PostgreSQL)** | Relational schema is the better fit for the Order–Payment–User transaction domain, but the product catalogue's heterogeneous specification data would require a JSONB column or EAV anti-pattern that negates the relational benefit. The team's stronger MongoDB proficiency reduces implementation risk within the sprint timeline. A hybrid approach (MongoDB for product catalogue, PostgreSQL for orders) was evaluated and rejected as over-engineering for an academic-scale system. |

### 4.2 React.js vs Vue.js

| Dimension | React.js 18.x | Vue.js 3.x |
|-----------|---------------|------------|
| **Learning curve** | Steeper (JSX, hooks mental model) | Gentler (Options API / Composition API, single-file components) |
| **Team familiarity** | All four members — direct coursework and project use | One of four members — self-study only |
| **Ecosystem** | Largest frontend ecosystem; most third-party library support | Smaller but growing ecosystem; fewer library options for niche integrations |
| **Performance** | Concurrent rendering (React 18), code-splitting, lazy loading | Reactivity system is efficient; comparable performance for typical CRUD |
| **Job market / community** | Dominant in Sri Lanka and global tech markets | Growing but smaller Sri Lanka developer community |
| **Component reuse** | JSX enables flexible composition patterns | Single-file `.vue` components are ergonomic but less portable to non-Vue contexts |
| **NFR alignment** | NFR-01 (code-splitting, Suspense) ✅ NFR-18 (large community → faster unblocking) ✅ | NFR-01 (Vue's reactivity is comparable) ✅ NFR-18 (smaller community → higher risk of blocked issues) ⚠️ |
| **Reason Rejected (Vue.js)** | Only one team member has any Vue experience. Switching to Vue would impose a learning curve on three engineers during a time-constrained academic project, directly threatening sprint delivery and the system availability goal of NFR-18. React's larger Sri Lanka developer community also provides faster access to local mentorship and community support. |

### 4.3 Tailwind CSS vs Bootstrap 5

| Dimension | Tailwind CSS 3.x (JIT) | Bootstrap 5.x |
|-----------|------------------------|---------------|
| **Approach** | Utility-first; compose styles in markup | Component-first; pre-built components with theme variables |
| **Bundle size** | JIT-purged: typically 5–15 KB in production | Full CSS: ~30 KB gzipped; tree-shaking available but less complete |
| **Customisation** | Design tokens in `tailwind.config.js` propagate everywhere; pixel-perfect control | Sass variable overrides; Bootstrap's visual language requires significant overriding for a custom brand |
| **Mobile-first** | Breakpoint prefixes (`sm:`, `md:`) natively in JSX; no media-query boilerplate | Responsive grid and utilities built-in; slightly less granular than Tailwind |
| **Accessibility** | Colour contrast enforced at token level; `focus-visible:` utilities built-in | Pre-built components have ARIA attributes; but contrast depends on theme colour choices |
| **Team familiarity** | Three of four members — direct project experience | All four members — coursework familiarity |
| **NFR alignment** | NFR-01 (smaller bundle) ✅ NFR-13 (mobile-first breakpoints) ✅ NFR-14 (token-level contrast enforcement) ✅ | NFR-01 (larger bundle) ⚠️ NFR-14 (contrast depends on overrides) ⚠️ |
| **Reason Rejected (Bootstrap)** | Bootstrap's pre-built component visual language (rounded cards, blue primary palette, modal styles) would require extensive Sass overriding to achieve the SmartTech design language, producing a larger final CSS bundle than Tailwind's JIT output. This conflicts with NFR-01's page-load budget. Three team members already use Tailwind, eliminating the tooling risk. |

### 4.4 REST API vs GraphQL

| Dimension | REST (JSON / HTTPS) | GraphQL |
|-----------|---------------------|---------|
| **Query flexibility** | Fixed endpoint per resource; over-fetching possible on complex pages | Single endpoint; client specifies exact fields; eliminates over-fetching |
| **Tooling** | `swagger-jsdoc` → OpenAPI 3.0 auto-generation; Postman; `curl` | GraphQL Playground / Apollo Studio; no OpenAPI equivalent |
| **Caching** | HTTP-level caching (CDN, browser) works natively | Requires Apollo Client normalised cache; HTTP caching non-trivial |
| **Learning curve** | Team knows REST natively | GraphQL schema definition, resolvers, and N+1 loader pattern are new to all members |
| **External integrations** | PayHere, SendGrid, AWS S3 — all REST; consistent end-to-end | External integrations are still REST; GraphQL only applies to internal API |
| **NFR alignment** | NFR-22 (stateless) ✅ NFR-25 (OpenAPI auto-gen) ✅ | NFR-22 ✅ NFR-25 (no OpenAPI; custom documentation required) ⚠️ |
| **Reason Rejected (GraphQL)** | The NFR-25 requirement for OpenAPI 3.0 documentation maps directly to REST with `swagger-jsdoc`. Implementing GraphQL would require a separate documentation strategy and introduce the N+1 query problem, which needs DataLoader to resolve — additional complexity with no user-visible benefit at SmartTech's scale. |

---

## 5. Full Technology Stack Summary

| Layer | Chosen Technology | Version | Primary NFR(s) | Named Alternative | Reason Alternative Rejected |
|-------|------------------|---------|----------------|-------------------|-----------------------------|
| Database | MongoDB | 7.x | NFR-02, NFR-23, NFR-24 | PostgreSQL 15 | Heterogeneous product specs require flexible schema; team familiarity lower for PostgreSQL |
| Backend runtime | Node.js | 20 LTS | NFR-03, NFR-28 | Python / Django | Shared JS across full stack; non-blocking I/O for I/O-bound marketplace workload |
| Backend framework | Express.js | 4.x | NFR-25, NFR-26 | Fastify | Express middleware ecosystem; team familiarity; swagger-jsdoc integration |
| Frontend framework | React.js | 18.x | NFR-01, NFR-18 | Vue.js 3 | 4/4 team proficiency; largest ecosystem; code-splitting for NFR-01 |
| Frontend styling | Tailwind CSS | 3.x | NFR-01, NFR-13, NFR-14 | Bootstrap 5 | JIT bundle size; mobile-first token system; WCAG contrast at token level |
| API style | REST (JSON / HTTPS) | — | NFR-22, NFR-25, NFR-28 | GraphQL | OpenAPI auto-generation; HTTP caching; consistent with all external integrations |

---

## 6. Consequences

### 6.1 Positive Consequences

- **Full-stack JavaScript.** A single language across frontend, backend, and database query layer (Mongoose/native driver) eliminates context switching. All four team members can contribute to any layer without language barriers.
- **Free-tier viable.** MongoDB Atlas M0, DigitalOcean App Platform (free tier), AWS S3 (5 GB free), and Netlify/Vercel (static hosting free) together keep hosting cost at £0 for development and early production.
- **Rapid UI development.** Tailwind JIT + React component composition enables new screens to be built and iterated within a single sprint, protecting the wireframe delivery milestones in Sprint 3.
- **OpenAPI compliance.** REST + `swagger-jsdoc` satisfies NFR-25 with near-zero overhead; the spec is auto-generated from existing route annotations rather than maintained as a separate file.

### 6.2 Negative Consequences / Trade-offs

- **MongoDB transaction ergonomics.** Multi-document ACID transactions (required for the atomic Order creation + stock reservation flow in UC-08) are supported since MongoDB 4.0 but are less idiomatic than PostgreSQL transactions. Mitigation: the `OrderService.create()` method wraps the entire order + stock decrement in a single `session.withTransaction()` block; unit tests verify rollback on failure.
- **Schema-less risk.** MongoDB's flexible schema can permit malformed documents if application-level validation is not enforced. Mitigation: Mongoose schemas with `strict: true` and `runValidators: true` enforce field types, required fields, and enum memberships at the ODM layer; `express-validator` provides a second validation gate at the API boundary.
- **React learning curve for complex state.** The seller dashboard's real-time order status updates and the admin analytics charts involve non-trivial state management. Mitigation: React Query (TanStack Query) handles server-state caching and background refetching, keeping component state logic simple.
- **Tailwind markup verbosity.** Utility-class strings on complex components can become long and hard to review in pull requests. Mitigation: shared components (e.g., `<Button variant="primary">`) abstract Tailwind class strings into a component library, keeping page-level JSX readable.

---

## 7. Compliance Checklist

| Acceptance Criterion | Status | Evidence |
|----------------------|--------|---------|
| AC1 — ADR-02 follows §7.2.1 template | ✅ Met | This document |
| AC2 — 6 technologies each have individual justification paragraph with named alternative | ✅ Met | Sections 3.1–3.6 |
| AC3 — Each technology's rationale references ≥ 1 NFR from SRS Ch. 5 | ✅ Met | NFRs cited in each §3.x and in §5 summary table |
| AC4 — Alternatives table covers MongoDB vs PostgreSQL, React vs Vue.js, Tailwind vs Bootstrap | ✅ Met | Sections 4.1, 4.2, 4.3 |
| AC5 — ADR-02 committed to `documents/architecture/decisions/ADR-02.md` | ⏳ In Progress | This file; commit pending PR review |

---

## 8. File Location

```
documents/
└── architecture/
    └── decisions/
        ├── ADR-01.md    ← Architectural pattern (Layered MVC + REST)
        └── ADR-02.md    ← Technology stack (MERN + Tailwind + REST)  ← this file
```

---

## 9. Review Sign-off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Author / Lead Architect | Piranavi Sasikaran (SE/2022/054) | 2026-05-10 | |
| Team Lead Review | | | |
| Supervisor Review | | | |

---

*Document conforms to: IEEE Std 1016-2009 · Course Guideline §7.2.1 · SmartTech SDS v1.0*
