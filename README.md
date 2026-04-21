# OpenLeap Developer Hub

**Start here.** This repository is the central entry point for the OpenLeap platform — an ERP product-line microservice construction kit.

---

## What is OpenLeap?

OpenLeap is a specification-driven ERP platform built on microservices. It provides:

- A **four-tier service architecture** for enterprise domains (finance, CRM, sales, HR, production, etc.)
- A **reusable service foundation** (Java 25 / Spring Boot 4, CQRS, event-driven)
- A **component-based UI platform** (Vue 3 / Nuxt 4, shared design system)
- An **AI-powered toolchain** (Agora: Elara + Telos + Noa) for discovery, specification, and code generation
- A **Product Line Engineering** approach for configuring customer-specific products from a shared feature catalog

For the SPLE platform model, see [architecture/sple-platform-concept.md](architecture/sple-platform-concept.md). For the full architecture overview, see [architecture/integrated-overview.md](architecture/integrated-overview.md).

---

## Repository Map

All repositories follow the naming pattern `io.openleap.{category}.{name}`. See [naming/repo-naming-strategy.md](naming/repo-naming-strategy.md) for the full convention.

### `dev.*` — Build It

Standards, specifications, templates, and tooling for developers.

| Repository | Purpose |
|------------|---------|
| **[dev.hub](https://github.com/openleap-io/io.openleap.dev.hub)** | This repo — central documentation & navigation |
| [dev.spec](https://github.com/openleap-io/io.openleap.dev.spec) | Domain & service specifications (T1-T4) |
| [dev.concepts](https://github.com/openleap-io/io.openleap.dev.concepts) | Specification framework — templates, artifacts, examples |
| [dev.prompts](https://github.com/openleap-io/io.openleap.dev.prompts) | AI automation prompts & scripts |
| [dev.guidelines](https://github.com/openleap-io/io.openleap.dev.guidelines) | Backend development guidelines & ADRs |
| [dev.parent](https://github.com/openleap-io/io.openleap.dev.parent) | Parent POM — Java 25, Spring Boot 4 |
| [dev.starter](https://github.com/openleap-io/io.openleap.dev.starter) | Spring Boot starter library (CQRS, outbox, security) |
| [dev.template](https://github.com/openleap-io/io.openleap.dev.template) | Microservice scaffold / template |
| [dev.archetype](https://github.com/openleap-io/io.openleap.dev.archetype) | Maven archetype for new services |
| [dev.bruno](https://github.com/openleap-io/io.openleap.dev.bruno) | API testing collections (Bruno) |
| [dev.customize](https://github.com/openleap-io/io.openleap.dev.customize) | Customization tooling |

### `ops.*` — Run It

Infrastructure, deployment, and CI/CD.

| Repository | Purpose |
|------------|---------|
| [ops.iac](https://github.com/openleap-io/io.openleap.ops.iac) | Docker Compose / Infrastructure as Code |
| [ops.k8s](https://github.com/openleap-io/io.openleap.ops.k8s) | Kubernetes manifests |
| [ops.workflows](https://github.com/openleap-io/io.openleap.ops.workflows) | Reusable GitHub Actions workflows |
| [ops.gateway](https://github.com/openleap-io/io.openleap.ops.gateway) | API Gateway |
| [ops.config](https://github.com/openleap-io/io.openleap.ops.config) | Spring Cloud Config Server |
| [ops.registry](https://github.com/openleap-io/io.openleap.ops.registry) | Eureka service discovery |

### `ui.*` — Show It

Shared UI libraries, design system, and standalone applications.

| Repository | Purpose |
|------------|---------|
| [ui.kit](https://github.com/openleap-io/io.openleap.ui.kit) | Component library (Vue 3, Shadcn-Vue, Tailwind 4) |
| [ui.base](https://github.com/openleap-io/io.openleap.ui.base) | Shared Nuxt layer (auth, shell, routing) |
| [ui.common](https://github.com/openleap-io/io.openleap.ui.common) | Common UI utilities |
| [ui.guidelines](https://github.com/openleap-io/io.openleap.ui.guidelines) | Frontend guidelines & ADRs |
| [ui.starter](https://github.com/openleap-io/io.openleap.ui.starter) | UI template project |
| [ui.admin](https://github.com/openleap-io/io.openleap.ui.admin) | Admin dashboard |
| [ui.elara](https://github.com/openleap-io/io.openleap.ui.elara) | Elara discovery UI |

### `ai.*` — Automate It

AI and automation tooling.

| Repository | Purpose |
|------------|---------|
| [ai.autoclaude](https://github.com/openleap-io/io.openleap.ai.autoclaude) | AI job runner |
| [ai.autoclaude.ui](https://github.com/openleap-io/io.openleap.ai.autoclaude.ui) | AI job runner UI |

### Business Domain Services

Organized by suite using the four-tier architecture.

#### T1 — Platform & Technical Foundations

| Suite | Domains |
|-------|---------|
| **IAM** | [iam.principal](https://github.com/openleap-io/io.openleap.iam.principal), [iam.authz](https://github.com/openleap-io/io.openleap.iam.authz), [iam.tenant](https://github.com/openleap-io/io.openleap.iam.tenant), [iam.audit](https://github.com/openleap-io/io.openleap.iam.audit) |
| **PARAM** | [t1.cfg](https://github.com/openleap-io/io.openleap.t1.cfg), [t1.i18n](https://github.com/openleap-io/io.openleap.t1.i18n), [t1.ref](https://github.com/openleap-io/io.openleap.t1.ref) |
| **TECH** | [tech.dms](https://github.com/openleap-io/io.openleap.tech.dms), [t1.jc](https://github.com/openleap-io/io.openleap.t1.jc), [t1_rpt](https://github.com/openleap-io/io.openleap.t1_rpt) |

#### T2 — Common (Cross-Suite Capabilities)

Two suites provide cross-suite capabilities consumed by every T3 business domain:

| Suite | Purpose | Domains |
|-------|---------|---------|
| **shared** | Cross-domain master data | [shared.bp](https://github.com/openleap-io/io.openleap.shared.bp) (Business Partner), [shared.cap](https://github.com/openleap-io/io.openleap.shared.cap) (Calendar & Planning) |
| **auto** | Cross-domain automation fabric | [auto.ntf](https://github.com/openleap-io/io.openleap.auto.ntf) (Notification Hub), [auto.wf](https://github.com/openleap-io/io.openleap.auto.wf) (Workflow Engine) *— being promoted from `crm.ntf` / `crm.wf`; 60-day routing-key bridge active* |

> `common.data` and `common.nfs` are **shared Java libraries**, not T2 domain services — see repo-catalog.yaml.

#### T3 — Core Business Suites

| Suite | Domains |
|-------|---------|
| **FI** (Finance) | [fi.coa](https://github.com/openleap-io/io.openleap.fi.coa), [fi.gl](https://github.com/openleap-io/io.openleap.fi.gl), [fi.ap](https://github.com/openleap-io/io.openleap.fi.ap), [fi.ar](https://github.com/openleap-io/io.openleap.fi.ar), [fi.acc](https://github.com/openleap-io/io.openleap.fi.acc), [fi.bank](https://github.com/openleap-io/io.openleap.fi.bank), [fi.fa](https://github.com/openleap-io/io.openleap.fi.fa), [fi.clr](https://github.com/openleap-io/io.openleap.fi.clr), [fi.cls](https://github.com/openleap-io/io.openleap.fi.cls), [fi.slc](https://github.com/openleap-io/io.openleap.fi.slc) |
| **CRM** | [crm.lead](https://github.com/openleap-io/io.openleap.crm.lead), [crm.contact](https://github.com/openleap-io/io.openleap.crm.contact), [crm.opp](https://github.com/openleap-io/io.openleap.crm.opp), [crm.act](https://github.com/openleap-io/io.openleap.crm.act), [crm.email](https://github.com/openleap-io/io.openleap.crm.email), and [13 more](https://github.com/orgs/openleap-io/repositories?q=io.openleap.crm) |
| **SD** (Sales & Distribution) | [sd.ord](https://github.com/openleap-io/io.openleap.sd.ord), [sd.bil](https://github.com/openleap-io/io.openleap.sd.bil), [sd.shp](https://github.com/openleap-io/io.openleap.sd.shp), [sd.dlv](https://github.com/openleap-io/io.openleap.sd.dlv), and [4 more](https://github.com/orgs/openleap-io/repositories?q=io.openleap.sd) |
| **PS** (Project Services) | [ps.prj](https://github.com/openleap-io/io.openleap.ps.prj), [ps.bud](https://github.com/openleap-io/io.openleap.ps.bud), [ps.tim](https://github.com/openleap-io/io.openleap.ps.tim), [ps.agl](https://github.com/openleap-io/io.openleap.ps.agl), [ps.prt](https://github.com/openleap-io/io.openleap.ps.prt), [ps.res](https://github.com/openleap-io/io.openleap.ps.res) |
| **SRV** (Service Delivery) | [srv.cas](https://github.com/openleap-io/io.openleap.srv.cas), [srv.apt](https://github.com/openleap-io/io.openleap.srv.apt), [srv.cat](https://github.com/openleap-io/io.openleap.srv.cat), and [4 more](https://github.com/orgs/openleap-io/repositories?q=io.openleap.srv) |
| **PUR** (Purchasing) | [pur.sup](https://github.com/openleap-io/io.openleap.pur.sup) |

---

## Architecture at a Glance

### Four-Tier Service Model

```
┌─────────────────────────────────────────────────────────────┐
│  T1 — Platform & Technical Foundations                       │
│  IAM, ref, i18n, si, dms, rpt, cfg, jc, search, email, ai   │
│  Suite-agnostic. Sync lookups. Invisible to business users. │
├─────────────────────────────────────────────────────────────┤
│  T2 — Common (Cross-Suite Capabilities)                      │
│  shared.{bp, cap}   — cross-suite master data (DDD kernel)   │
│  auto.{ntf, wf}     — cross-suite automation fabric          │
├─────────────────────────────────────────────────────────────┤
│  T3 — Core Business Suites                                   │
│  FI, CRM, SD, PS, SRV, PPS, OPS, CO, COM, FAC, HR, TKS      │
│  Suite-specific bounded contexts. Domain events.             │
├─────────────────────────────────────────────────────────────┤
│  T4 — Data, Analytics & Integration                          │
│  BI, ETL pipelines, data lake                                │
│  Read-only consumer of all domain events.                    │
└─────────────────────────────────────────────────────────────┘
```

For the complete architecture with event flows and Mermaid diagrams, see [architecture/system-topology.md](architecture/system-topology.md).

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Java 25, Spring Boot 4, PostgreSQL 16, RabbitMQ 4.x |
| Frontend | Vue 3, Nuxt 4, TypeScript, Tailwind 4, Shadcn-Vue |
| AI | Anthropic Claude, Spring AI 2.0, MongoDB, Milvus, Neo4j |
| Auth | Keycloak 26.x, OPA |
| Infra | Kubernetes, Docker, GitHub Actions |

### Communication Patterns

| Pattern | Convention | Example |
|---------|-----------|---------|
| REST API | `/api/<suite>/<domain>/v1` | `/api/fi/gl/v1/journals`, `/api/auto/ntf/v1/notifications` |
| Event exchange | `<suite>.<domain>.events` | `pps.pd.events`, `shared.bp.events` |
| Routing key | `<suite>.<domain>.<aggregate>.<event>` | `pps.pd.product.released`, `auto.wf.workflow.triggered` |
| DB schema | `<suite>_<domain>` | `pps_pd`, `shared_bp`, `auto_notification` |
| Java package | `io.openleap.<suite>.<domain>` | `io.openleap.fi.gl`, `io.openleap.auto.ntf` |

---

## Documentation in This Repo

| Path | Content |
|------|---------|
| [naming/](naming/) | Repository naming strategy and conventions |
| [architecture/](architecture/) | Platform architecture overview and system topology |
| [landscape/](landscape/) | Ecosystem status, repo catalog, integration maps |

### Naming

- [naming/repo-naming-strategy.md](naming/repo-naming-strategy.md) — How repos are named: `dev`, `ops`, `ui`, `ai` + domain infixes

### Architecture

- [architecture/sple-platform-concept.md](architecture/sple-platform-concept.md) — SPLE model: Platform Engineering, Application Engineering, and the specification artifacts that bridge them
- [architecture/product-repo-layout.md](architecture/product-repo-layout.md) — `io.openleap.prod.{product}` repository convention
- [architecture/getting-started-product.md](architecture/getting-started-product.md) — From product idea to release, with CRM Workbench and Ticket Desk worked examples
- [architecture/integrated-overview.md](architecture/integrated-overview.md) — Integrated architecture overview (data, communication, AI tooling)
- [architecture/four-tier-architecture.md](architecture/four-tier-architecture.md) — Four-tier architecture, suites, communication patterns
- [architecture/t4-structure.md](architecture/t4-structure.md) — T4 Data, Analytics & Data Activation: three pillars, SPLE split, integration decomposition
- [architecture/system-topology.md](architecture/system-topology.md) — System topology with Mermaid diagrams

### Landscape

- [landscape/repo-landscape.md](landscape/repo-landscape.md) — Repository inventory and dependency graph
- [landscape/repo-catalog.yaml](landscape/repo-catalog.yaml) — Machine-readable repository metadata
- [landscape/integration-map.md](landscape/integration-map.md) — Platform integration flows and API contract mappings
- [landscape/spec-status.json](landscape/spec-status.json) — Spec completeness per domain/service
- [landscape/impl-status.json](landscape/impl-status.json) — Implementation status across repos

### Diagrams

- [landscape/diagrams/conceptual-architecture.mmd](landscape/diagrams/conceptual-architecture.mmd)
- [landscape/diagrams/repo-dependencies-backend.mmd](landscape/diagrams/repo-dependencies-backend.mmd)
- [landscape/diagrams/repo-dependencies-frontend.mmd](landscape/diagrams/repo-dependencies-frontend.mmd)
- [landscape/diagrams/specification-lifecycle.mmd](landscape/diagrams/specification-lifecycle.mmd)
- [landscape/diagrams/tier-layer-reconciliation.mmd](landscape/diagrams/tier-layer-reconciliation.mmd)

---

## Getting Started

### New to OpenLeap?

1. Read this README for the high-level picture
2. Read [architecture/integrated-overview.md](architecture/integrated-overview.md) for the full architecture
3. Browse the [naming strategy](naming/repo-naming-strategy.md) to understand repo organisation
4. Check [landscape/repo-landscape.md](landscape/repo-landscape.md) for the full repo inventory

### Building a new service?

1. Read the [dev.guidelines](https://github.com/openleap-io/io.openleap.dev.guidelines) for backend standards
2. Check the [dev.spec](https://github.com/openleap-io/io.openleap.dev.spec) for your domain's specification
3. Use the [dev.template](https://github.com/openleap-io/io.openleap.dev.template) or [dev.archetype](https://github.com/openleap-io/io.openleap.dev.archetype) to scaffold
4. Reference [dev.concepts](https://github.com/openleap-io/io.openleap.dev.concepts) for specification templates

### Designing a product?

1. Read [architecture/sple-platform-concept.md](architecture/sple-platform-concept.md) for the two-lifecycle model
2. Walk through [architecture/getting-started-product.md](architecture/getting-started-product.md) — end-to-end worked examples (CRM Workbench + Ticket Desk)
3. Use [architecture/product-repo-layout.md](architecture/product-repo-layout.md) for the `io.openleap.prod.{product}` repo shape
4. Author in Elara; export to a `prod.*` repo for review and release

### Working on the UI?

1. Read the [ui.guidelines](https://github.com/openleap-io/io.openleap.ui.guidelines) for frontend standards
2. Use [ui.kit](https://github.com/openleap-io/io.openleap.ui.kit) for components
3. Extend [ui.base](https://github.com/openleap-io/io.openleap.ui.base) for shared layers
4. Start from [ui.starter](https://github.com/openleap-io/io.openleap.ui.starter) for new product UIs

### Deploying?

1. Check [ops.k8s](https://github.com/openleap-io/io.openleap.ops.k8s) for Kubernetes manifests
2. Use [ops.workflows](https://github.com/openleap-io/io.openleap.ops.workflows) for CI/CD
3. See [ops.iac](https://github.com/openleap-io/io.openleap.ops.iac) for local Docker Compose setup
