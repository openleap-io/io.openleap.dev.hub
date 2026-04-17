# OpenLeap — Integrated Architecture Overview (Gesamtsicht)

Last updated: 2026-04-01

This document is the single entry point for understanding the complete OpenLeap ecosystem — its conceptual model, platform products, service architecture, repository structure, and specification system. It synthesizes information from [`dev.spec`](https://github.com/openleap-io/io.openleap.dev.spec), [`dev.concepts`](https://github.com/openleap-io/io.openleap.dev.concepts), and the [landscape](../landscape/) directory.

---

## 1. Platform Identity

**OpenLeap** is an ERP product-line microservice construction kit (_Produktlinien-Microservice-Baukasten_). It provides:

- A **specification-driven architecture** for enterprise domains (finance, production, sales, HR, etc.)
- A **reusable service foundation** (Java 25 / Spring Boot 4, CQRS, event-driven)
- A **component-based UI platform** (Vue 3 / Nuxt 4, shared design system)
- An **AI-powered discovery and architecture toolchain** (Agora: Elara + Telos + Noa)
- A **Product Line Engineering** approach for configuring customer-specific products from a shared feature catalog

The platform distinguishes three fundamental perspectives:

| Perspective | Question | Tool | Key Artifacts |
|-------------|----------|------|---------------|
| **Problem Space** | "What does the business need?" | Elara | Processes, actors, rules, glossary, domains |
| **Solution Space** | "How is it built?" | Telos | Suite specs, service specs, workflow specs, contracts |
| **Product Space** | "What does the user interact with?" | Elara (config) + Telos (catalog) + UI-SPLE (variability) | Products, features, personas, screen contracts |

---

## 2. Agora — The Meta-Platform

**Agora** (ἀγορά — marketplace) is the AI-powered toolchain that sits above the ERP services. It orchestrates the entire lifecycle from business discovery to technical specification.

```
        ELARA                          TELOS                          NOA
  "Discover what to build"     "Architect how to build it"    "The mind behind it"
  ─────────────────────────    ───────────────────────────    ─────────────────────
  AI-guided interviews         Suite & Service Specs          Shared AI agent
  BPMN process modeling        Feature & Workflow Specs       Claude-powered
  Domain clustering            Registry & Dependency Graph    5 operation modes
  Product configuration        Semantic search (Milvus)       Platform-filtered tools
  Persona management           Impact analysis (Neo4j)        Context-aware prompts

           │                            ▲
           └── Conceptual Freeze ──────┘
               (formal handoff)
```

### The Elara → Telos Pipeline

1. **Discover** (Elara) — AI-guided interviews extract processes, actors, glossary, business rules
2. **Compose** (Elara) — Structure findings into domains, identify products and personas
3. **Specify** (Elara) — Refine processes, select features from Telos catalog
4. **Engineer** (Elara) — Produce Conceptual Freeze artifact
5. **Decompose** (Telos) — Import freeze, match against catalog, identify gaps
6. **Specify** (Telos) — Create/update suite specs, service specs, feature specs, workflow specs

---

## 3. The Unified Model

OpenLeap uses three orthogonal models that are frequently referenced together. Their formal reconciliation is documented in [`dev.concepts/RECONCILIATION.md`](https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/RECONCILIATION.md).

### 3.1 Four-Tier Service Model (Runtime)

Governs deployment topology and dependency direction. Higher tiers may depend on lower tiers but not vice versa.

```
┌─────────────────────────────────────────────────────────────┐
│  T1 — Platform & Technical Foundations                       │
│  IAM, ref, i18n, si, dms, rpt                               │
│  Suite-agnostic. Sync lookups. Invisible to business users.  │
├─────────────────────────────────────────────────────────────┤
│  T2 — Shared Enterprise Business                             │
│  bp (Business Partner), cal (Calendar)                       │
│  Cross-suite master data. DDD shared kernel.                 │
├─────────────────────────────────────────────────────────────┤
│  T3 — Core Business Suites                                   │
│  PPS (10 domains), FI (6), SD (3), HR (2), PS (1)           │
│  Suite-specific bounded contexts. Domain events.             │
│  Feature mutations stay within ONE suite.                    │
├─────────────────────────────────────────────────────────────┤
│  T4 — Data, Analytics & Integration                          │
│  BI, ETL pipelines, data lake                                │
│  Read-only consumer of all domain events.                    │
└─────────────────────────────────────────────────────────────┘
```

See [`system-topology.md`](system-topology.md) for the complete domain map and event flow diagram.

### 3.2 Five-Layer Specification Model (Authoring)

Governs what kind of knowledge an artifact captures. The **Perspektivwechsel** (perspective break) sits between Conceptual and Suite/System layers.

| Layer | Question | Platform | Artifacts |
|-------|----------|----------|-----------|
| Business Process | "What happens?" | Elara | BPMN, actors, rules, KPIs |
| Conceptual | "What do we need?" | Elara → Freeze | Domains, requirements, service boundary candidates |
| Suite (logical) | "What can the domain do?" | Telos | UBL, domain model, service landscape, ADRs |
| System (technical) | "How does it run?" | Telos | Infra specs, tech ADRs, deployment configs |
| Service | "How is it built?" | Telos | API contracts, events, entities, data model |

Suite and System are orthogonal at the same depth — logical vs. technical perspective.

### 3.3 Three-Space Perspective (Lifecycle)

| Space | Owner | When | Stable? |
|-------|-------|------|---------|
| Problem Space | Elara | Discovery & analysis | High — business reality doesn't change with refactoring |
| Solution Space | Telos | Architecture & specification | Medium — changes with implementation decisions |
| Product Space | Elara + Telos + UI-SPLE | Configuration & derivation | Low — changes per customer/project |

---

## 4. Specification Taxonomy

All specifications follow a clear hierarchy. Each type has a defined template and location.

```
Suite Spec                    (one per T3 suite — e.g., PPS, FI, SD)
  │  Defines: scope, UBL, domain model, service landscape, event conventions
  │  Location: spec/T3_Domains/{SUITE}/_*_suite.md
  │
  ├── Service Spec            (one per microservice within a suite)
  │     Defines: aggregates, invariants, REST API, events, data model, security
  │     Location: spec/T3_Domains/{SUITE}/{suite}_{domain}.md
  │
  ├── Feature Spec            (one per user-facing capability, owned by suite)
  │     Defines: user goal, journey, screens, backend deps, acceptance criteria
  │     Location: Telos catalog (F-{SUITE}-{NNN}.md)
  │     Machine-readable: UI-SPLE (F-{SUITE}-{NNN}.uvl)
  │
  └── Workflow Spec           (one per non-actor process — batch, saga, ETL)
        Defines: trigger, steps, compensation, retry, observability
        Location: Telos catalog (wf-{name})
```

**Contracts** are derived from specs and maintained in the respective service implementation repositories. Contract conventions (OpenAPI, AsyncAPI, JSON Schema) are defined in the spec templates under `concepts/templates/`.

---

## 5. Data Architecture

The platform uses two distinct data strategies depending on the service type.

### 5.1 ERP Domain Services (T1–T4)

| Concern | Technology | Pattern |
|---------|-----------|---------|
| Persistence | PostgreSQL 16 | One DB per service, schema `<suite>_<domain>` |
| Messaging | RabbitMQ | Transactional outbox → topic exchanges, at-least-once |
| Caching | Caffeine (local) / Redis (distributed) | Cache-aside with event-based invalidation |
| Search | PostgreSQL full-text | Per-service, no external search engine |

**CQRS pattern:** Separate write and read models. Commands go through CommandGateway → CommandBus → CommandHandler → Aggregate → Outbox. Queries use dedicated read repositories with denormalized projections.

### 5.2 Agora Platform Services (Elara, Telos, Noa)

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Structured | MongoDB | CRUD for specs, processes, conversations, artifacts |
| Semantic | Milvus | Vector search across spec chunks, chat context, "between the lines" knowledge |
| Relational | Neo4j | Dependency graphs, impact analysis, transitive path finding |

Agora services do not use PostgreSQL or RabbitMQ — they have their own data strategy optimized for knowledge management rather than transactional processing.

---

## 6. Communication Architecture

### 6.1 Synchronous (REST)

Used for data lookups and transactional commands.

| Convention | Pattern | Example |
|-----------|---------|---------|
| Base path | `/api/<suite>/<domain>/v1` | `/api/pps/pd/v1` |
| Commands | `POST` returning `202 Accepted` | `POST /api/fi/gl/v1/journals` |
| Queries | `GET` with pagination | `GET /api/shared/bp/v1/parties?page=0&size=50` |
| Concurrency | ETag + `If-Match` | Optimistic locking on aggregate version |
| T1 lookups | Sync REST from any tier | REF for codes, SI for units, DMS for documents |

### 6.2 Asynchronous (RabbitMQ Events)

Used for domain change notifications. Decouples services and enables eventual consistency.

| Convention | Pattern | Example |
|-----------|---------|---------|
| Exchange | `<suite>.<domain>.events` (topic) | `pps.pd.events` |
| Routing key | `<suite>.<domain>.<aggregate>.<event>` | `pps.pd.product.released` |
| Consumer queue | `<consumer>.<consumer-domain>.in.<suite>.<domain>.events` | `pps.mrp.in.pps.pd.events` |
| Delivery | At-least-once, idempotent consumers | CommandId + natural keys for dedup |
| Payload | Thin events (IDs + change metadata) | Consumers fetch details via REST |
| Publishing | Transactional outbox pattern | Same TX as domain mutation |

### 6.3 Key Event Flows

```
pps.pd.product.released  → pps.mrp, pps.pur, pps.mes, pps.qm, pps.im
pps.pur.po.created       → pps.im (goods receipt expectation)
pps.im.goodsreceipt.posted → pps.pur (3-way match), pps.qm (quality check)
sd.sd.delivery.created   → pps.im (goods issue)
sd.sd.billing.created    → fi.ar (financial posting)
fi.ap.invoice.posted     → pps.pur (invoice matching)
bp.bp.party.created      → pps.pur, sd.sd, hr.hr, fi.ap (master data sync)
all domain events        → T4 bi (analytics ingestion)
```

---

## 7. AI Architecture — Noa

Noa is the shared AI intelligence layer powered by Anthropic Claude. It belongs to neither Elara nor Telos but serves both.

**Operation modes:**

| Mode | Temp | Use Case |
|------|------|----------|
| EXPLORE | 0.9 | Open brainstorming, pattern discovery |
| ANALYZE | 0.4 | Structured queries against registry + graph |
| GENERATE | 0.5 | Spec section drafting, ADR creation |
| REVIEW | 0.1 | Validation, completeness checks |
| EXTRACT | 0.1 | Document segmentation, conflict detection (AIR pipeline) |

**Tool categories:** Elara tools (process_*, actor_*, glossary_*, product_*, persona_*), Telos tools (registry_*, spec_*, graph_*, feature_*, workflow_*), Shared tools (web_*, doc_*, air_*).

**System prompt assembly:** 6 blocks dynamically composed per request — Role & Identity, Operation Mode, Scope Context, Tool Descriptions, Behavior Rules, Conversation History.

---

## 8. Product Line Engineering (PLE)

OpenLeap implements PLE through three complementary mechanisms:

### 8.1 Feature Catalog (Telos — Domain Engineering)

Features are **owned by suites**, not products. The catalog is project-independent and grows over time.

```
Platform Catalog (catalog.uvl)
├── sd.catalog.uvl
│   ├── F-SD-001 Create Order (composition node)
│   │   ├── F-SD-001-01 Order Overview (leaf)
│   │   └── F-SD-001-02 Order Entry (leaf)
│   └── F-SD-002 ...
├── fi.catalog.uvl
│   └── ...
└── bp.catalog.uvl
    └── ...
```

### 8.2 Product Configuration (Elara — Application Engineering)

Products are assembled in Elara by selecting features from the catalog and resolving variability. The resulting specification is exported to a Git-backed product repository (`io.openleap.prod.{product}`) for review, versioning, and release — Elara is the authoring UI, the repo is the system of record. See [product-repo-layout.md](../architecture/product-repo-layout.md) for the full repository convention.

| Concept | Definition |
|---------|-----------|
| Product | Composition of platform features serving one or more personas. May span multiple suites (e.g. a Sales Workbench drawing from SD, BP, FI, PPS). |
| Product spec repo | `io.openleap.prod.{product}` — Git-tracked ProductConfig, extensions, personas, processes, screen overrides. |
| Feature inclusion | `full`, `read-only`, `embedded`, or `excluded` |
| Cross-suite features | T2 features freely included. Other suite's T3 features only `read-only`. |
| Variability resolution | Attributes set per binding time: `compile`, `deploy`, `runtime` |

### 8.3 Screen Contracts (UI-SPLE — Product Derivation)

UI-SPLE applies the Cameleon Reference Framework with a pragmatic two-layer approach:

| Layer | Purpose | Platform |
|-------|---------|----------|
| **AUI** (Abstract UI) | Platform-free specification: tasks, zones, priorities, business rules, feature flags | Shared across platforms |
| **CUI** (Concrete UI) | Platform-specific: responsive breakpoints (web), touch navigation (mobile), keyboard-first (desktop) | Per platform |

**Zone types:** Fixed (base.ui owns), Variable (product team slots), Feature-gated (optional via feature flags).

**Absent-rules:** `collapse-up`, `end-page`, `tab-hidden`, `panel-section-hidden` — define what happens when optional zones are not rendered.

---

## 9. Technical Landscape

### 9.1 Repository Ecosystem

39 repositories organized in 8 categories. For the complete inventory and dependency graphs, see [`landscape/repo-landscape.md`](../landscape/repo-landscape.md).

```
io.openleap.spec                    ← You are here (specifications)
  ↑ defines WHAT
  │
io.openleap.dev.guidelines          ← Backend rules (2137 rules, 63 ADRs)
io.openleap.ui.guidelines           ← Frontend rules (261 rules, 31 ADRs)
  ↑ define HOW
  │
io.openleap.parent                  ← Maven BOM (Java 25, Spring Boot 4.0.2)
io.openleap.starter                 ← Core service library (CQRS, outbox, security)
  ↑ provide FOUNDATION
  │
io.openleap.{suite}.{domain}        ← Backend services (T1–T4 + Agora)
io.openleap.ui.kit                  ← Component library (6 packages)
io.openleap.ui.base                 ← Shared Nuxt layer (auth, shell)
io.openleap.{product}.ui            ← Product frontends
```

### 9.2 Backend Stack

| Component | Version | Governance |
|-----------|---------|------------|
| Java | 25 | `io.openleap.parent` |
| Spring Boot | 4.0.2 | `io.openleap.parent` |
| Spring Cloud | 2025.1.1 | `io.openleap.parent` |
| PostgreSQL | 16 | Per service |
| RabbitMQ | 4.x | Per service |
| Keycloak | 26.x | Platform-wide |

Agora services additionally use: MongoDB, Milvus 2.4, Neo4j 5.x, Spring AI 2.0.

### 9.3 Frontend Stack

| Component | Version | Governance |
|-----------|---------|------------|
| Vue | 3.5+ | `io.openleap.ui.kit` |
| Nuxt | 4.x | Per product UI |
| TypeScript | 5.7–5.9 | Per product UI |
| Tailwind CSS | 4 | `io.openleap.ui.kit` |
| Shadcn-Vue | Latest | `io.openleap.ui.kit` |
| TanStack Query/Table | Latest | `io.openleap.ui.base` |

### 9.4 AI & Automation

| Tool | Purpose |
|------|---------|
| Anthropic Claude API | Noa (AI layer), Telos AI (spec analysis) |
| Claude Code CLI | `nightagent` (autonomous sandbox), `ai.autoclaude` (job runner) |
| Spring AI 2.0 | LLM integration in Java services |

---

## 10. Governance

### 10.1 Specification Authority

`io.openleap.spec/spec/` is the **canonical source** for all domain specifications. Service repos may contain derived OpenAPI specs for development convenience, but the spec repo takes precedence on conflicts.

### 10.2 Dependency Rules

- T3 services may depend on T1 and T2 (sync reads or event consumption)
- T3 services must NOT depend directly on other T3 suites (only via events or T2 abstractions)
- T4 is a read-only consumer — it never produces domain events
- Feature mutations must stay within the owning suite's T3 services

Governance rules are documented in the dev guidelines repo (`io.openleap.dev.guidelines`).

### 10.3 Naming Conventions

| Aspect | Pattern | Example |
|--------|---------|---------|
| Exchange | `<suite>.<domain>.events` | `pps.pd.events` |
| Routing key | `<suite>.<domain>.<aggregate>.<event>` | `pps.pd.product.released` |
| API path | `/api/<suite>/<domain>/v1` | `/api/fi/gl/v1` |
| DB schema | `<suite>_<domain>` | `pps_pd` |
| Java package | `io.openleap.<suite>.<domain>` | `io.openleap.fi.gl` |
| Feature ID | `F-{SUITE}-{NNN}[-{NN}]` | `F-SD-001-02` |

---

## 11. Glossary

For the complete unified glossary covering all conceptual models, see [`dev.concepts/RECONCILIATION.md` §3](https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/RECONCILIATION.md).

Key terms quick reference:

| Term | Definition |
|------|-----------|
| **Tier** | Runtime deployment layer (T1–T4) |
| **Layer** | Specification abstraction level (5-Layer model) |
| **Space** | Lifecycle perspective (Problem/Solution/Product) |
| **Suite** | Logical grouping of T3 domain services. Owns features. |
| **Domain** | Bounded context within a suite. Maps to one microservice. |
| **Product** | Composition of platform features serving one or more personas. May span multiple suites. Spec lives in `io.openleap.prod.{product}`. |
| **Feature** | Suite-owned capability. Three views: Candidate (Elara) → Spec (Telos) → UVL (UI-SPLE) |
| **Conceptual Freeze** | Formal handoff artifact from Elara to Telos |

---

## Navigation

| What you need | Where to go |
|--------------|-------------|
| Domain specifications | [`dev.spec`](https://github.com/openleap-io/io.openleap.dev.spec) — organized by tier |
| Conceptual frameworks | [`dev.concepts`](https://github.com/openleap-io/io.openleap.dev.concepts) — templates, artifacts, reconciliation |
| Conceptual stack | [`dev.concepts/CONCEPTUAL_STACK.md`](https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/CONCEPTUAL_STACK.md) — Agora platform model |
| Spec templates | [`dev.concepts/templates/`](https://github.com/openleap-io/io.openleap.dev.concepts/tree/main/templates) — specification authoring templates |
| Term definitions | [`dev.concepts/RECONCILIATION.md`](https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/RECONCILIATION.md) — unified glossary |
| Repository inventory | [`landscape/repo-landscape.md`](../landscape/repo-landscape.md) |
| Repository metadata | [`landscape/repo-catalog.yaml`](../landscape/repo-catalog.yaml) |
| Backend dev guidelines | [`dev.guidelines`](https://github.com/openleap-io/io.openleap.dev.guidelines) |
| Frontend dev guidelines | [`ui.guidelines`](https://github.com/openleap-io/io.openleap.ui.guidelines) |
