# OpenLeap ERP Platform — General Documentation

> **Origin:** migrated from `io.openleap.spec/spec/OPENLEAP_PLATFORM_GENERAL.md` (last substantive update 2026-04-01; moved to dev.hub/architecture/ 2026-04-21).
> **Companion:** [`four-tier-architecture.md`](four-tier-architecture.md) is the primary reference; this document adds depth on communication patterns, governance, and reliability.

This document provides a concise, product-agnostic overview of the OpenLeap ERP platform. It explains the four-tier architecture, outlines typical suites and domains in each tier, and describes the platform-wide communication patterns: synchronous APIs for data access and asynchronous eventing (RabbitMQ) for business-change notifications.


## 1. Four-Tier Architecture

OpenLeap adopts a pragmatic four-tier architecture to separate technical foundations, cross-suite capabilities, core business execution, and analytical/integration concerns.

- Tier 1 — Platform & Technical Foundations (T1)
  - Purpose: Provide shared, low-level platform services and reference data needed across the enterprise. These services are highly stable, read-optimized, and accessed synchronously by higher tiers.
  - Characteristics: Small, focused services; strong backward compatibility; high availability; multi-tenant aware (where applicable).

- Tier 2 — Common (T2)
  - Purpose: Cross-suite capabilities consumed by every T3 domain. Organised into two suites:
    - **shared** — enterprise master data (Business Partner, Calendar & Planning). Authoritative sources; lifecycle events published for downstream consumers.
    - **auto** — automation fabric (Notification Hub, Workflow Engine). Declarative rules, multi-channel delivery, assignment & escalation.
  - Characteristics: Suite-agnostic kernel services; consistent identifiers; domain-independent primitives that T3 domains extend with their own semantics.

- Tier 3 — Core Business Suites (T3)
  - Purpose: Deliver the operational core of the ERP. Organized into suites (e.g., PPS, FI, SD, TKS), each containing multiple business domains.
  - Characteristics: Rich domain models; orchestrate end-to-end processes; emit domain events; depend on T1 synchronously and on T2/T3 asynchronously where appropriate.

- Tier 4 — Data, Analytics & Integration (T4)
  - Purpose: Provide data platform, analytics, and external integration capabilities. Consumes events from all tiers and exposes curated data products.
  - Characteristics: Event ingestion at scale; schema-on-write/read per use case; minimal coupling to operational workflows.


## 2. Typical Suites and Domains per Tier

Below are representative (non-exhaustive) suites and domains used across OpenLeap. Actual deployments can tailor these lists.

### 2.1 Tier 1 — Platform & Technical Foundations (T1)
- ref — Reference Data (codes, classifications, catalogs)
- i18n — Internationalization (labels, translations)
- si — Units of Measure (SI units, conversions)
- dms — Document Management (file storage, metadata, rendition)
- rpt — Technical Reporting Utilities (PDF rendering and templates)
- cfg — Platform Configuration (feature flags, application settings)
- jc — Job Control (async job scheduling and monitoring)
- search — Platform Search (promoted from `crm.search`)
- email — Platform Email (promoted from `crm.email`)
- ai — LLM & Embedding abstraction (new)
- iam — Identity & Access Management (principal, tenant, authz, audit)

Access pattern: Primarily synchronous lookups from higher tiers (e.g., resolve codes, labels, units, templates). These services may also emit update events when reference data changes, but most consumers rely on on-demand reads.

### 2.2 Tier 2 — Common (T2)

Two cross-suite suites:

**shared** — master data
- bp — Business Partner (parties, organizations, contacts, relationships)
- cap — Calendar & Planning (calendars, working patterns, holidays, slots, bookings, periods)

**auto** — automation fabric *(promoted from legacy CRM domains; 60-day routing-key bridge active)*
- ntf — Notification Hub (multi-channel delivery, per-principal preferences, templates, digest & quiet hours) — supersedes `crm.ntf`
- wf — Workflow Engine (triggers, conditions, actions, assignment & escalation rules, template library) — supersedes `crm.wf`

Access pattern: Authoritative cross-suite services with both synchronous APIs and asynchronous lifecycle events (e.g., `shared.bp.party.created`, `auto.wf.workflow.triggered`).

### 2.3 Tier 3 — Core Business Suites (T3)

Suites group cohesive operational domains. Common suites include:

- PPS — Production, Planning & Shopfloor
  - pd — Product Definition (materials, BOM, routings, production versions)
  - mrp — Material Requirements Planning (supply proposals)
  - mes — Manufacturing Execution (confirmations, shopfloor operations)
  - aps — Advanced Planning & Scheduling
  - qm — Quality Management
  - im — Inventory Management (stock, goods movements)
  - wm — Warehouse Management (picking, putaway, waves)
  - pur — Procurement (suppliers, purchase orders, ASN)
  - eam — Enterprise Asset Management
  - co — Controlling (if hosted within PPS in a given deployment)

- FI — Finance & Accounting
  - gl — General Ledger
  - ap — Accounts Payable
  - ar — Accounts Receivable
  - tax — Tax
  - pay — Payments
  - co — Controlling (if hosted within FI in a given deployment)

- SD — Sales & Customer Management
  - sd — Sales Core (orders, deliveries, billing)
  - pricing — Pricing
  - crm — CRM (note: `crm.ntf`, `crm.wf`, `crm.search`, `crm.email`, `crm.sup` are DEPRECATED — see promotion notes)

- TKS — Ticket System (new T3 suite as of April 2026)
  - tkt — Tickets
  - ch — Channels
  - kb — Knowledge Base
  - cmdb — Configuration Items

- HR — Human Resources
  - hr — Core HR
  - time — Time & Attendance

- PS — Project Management
  - ps — Projects (WBS, activities)

- CO — Controlling
  - abc — Activity-Based Costing
  - cca — Cost Center Accounting
  - io — Internal Orders
  - om — Overhead Management
  - pa — Profitability Analysis
  - pc — Product Costing
  - pca — Profit Center Accounting
  - rpt — Reporting

- COM — Commerce
  - cat — Catalog
  - cmp — Campaigns
  - lst — Listings
  - mpx — Marketplace Exchange
  - prc — Pricing
  - srch — Search

- CRM — Customer Relationship Management
  - contact — Contact Management
  - lead — Lead Management
  - opportunity — Opportunities
  - activity — Activities
  - (deprecated domains being promoted: `crm.ntf` → `auto.ntf`, `crm.wf` → `auto.wf`, `crm.search` → `tech.search`, `crm.email` → `tech.email`, `crm.sup` → `tks.tkt` + `tks.kb`)

- FAC — Facilities
  - col — Collections
  - fnd — Foundations
  - lim — Limits
  - rcv — Receivables
  - rsk — Risk

- OPS — Operations / Field Service
  - doc — Documents
  - ord — Orders
  - prj — Projects
  - res — Resources
  - sch — Scheduling
  - svc — Services
  - tim — Time Tracking
  - tsk — Tasks

- SRV — Service Management
  - apt — Appointments
  - bil — Billing
  - cas — Cases
  - cat — Catalog
  - ent — Entitlements
  - res — Resources
  - ses — Sessions

Access pattern: Rich domain APIs (synchronous) and high-value business events (asynchronous) published per domain.

### 2.4 Tier 4 — Data, Analytics & Integration (T4)
- bi — Data Platform (data ingestion, curation, analytics serving)

Access pattern: Primarily event-driven ingestion from all tiers; may provide HTTP APIs for data products and governed extracts.


## 3. Naming Conventions and Endpoints

To ensure interoperability and discoverability, OpenLeap follows consistent naming for APIs and events:

- HTTP API base paths: `/api/<suite>/<domain>/v1`
  - Examples: `/api/pps/pd/v1`, `/api/fi/ap/v1`, `/api/shared/bp/v1`, `/api/auto/ntf/v1`, `/api/t1/ref/v1`

- Event exchanges (RabbitMQ topic exchanges): `<suite>.<domain>.events`
  - Examples: `pps.pd.events`, `fi.ap.events`, `shared.bp.events`, `auto.ntf.events`, `auto.wf.events`, `i18n.i18n.events`

- Routing keys: `<suite>.<domain>.<aggregate>.<event>`
  - Examples: `pps.pd.product.released`, `pps.im.goodsreceipt.posted`, `sd.sd.billing.created`, `fi.ap.invoice.posted`, `shared.bp.party.created`, `auto.wf.workflow.triggered`, `auto.ntf.notification.delivered`


## 4. Communication Patterns

OpenLeap combines synchronous communication for data access with asynchronous messaging for change notifications and cross-domain process choreography.

### 4.1 Synchronous communication — for data lookup and transactional APIs
- When to use:
  - Read reference/master data on demand (e.g., resolve code → label via T1 `ref`, fetch units via T1 `si`, look up Business Partner via T2 `shared.bp`).
  - Execute transactional commands within a domain boundary (CRUD as allowed, posting movements, confirming operations).
  - Retrieve current state snapshots (e.g., stock by location, BP profile).
- Protocols: HTTP/REST with JSON as default; gRPC may be used for internal high-throughput cases (optional).
- Semantics:
  - Request-response; clients handle retries with exponential backoff.
  - Idempotency keys recommended for mutating operations.
  - Strong validation and clear error models (4xx client errors, 5xx server errors).

### 4.2 Asynchronous communication — for events and notifications (RabbitMQ)
- When to use:
  - Publish domain events representing facts that happened (e.g., product released, PO created, goods receipt posted, invoice posted, party created).
  - Notify interested domains in other suites; enable eventual consistency and decoupled reactions.
  - Trigger cross-suite automation via `auto.wf` workflows.
  - Feed Tier 4 BI for analytics at scale.
- Infrastructure: RabbitMQ topic exchanges per domain: `<suite>.<domain>.events`.
- Contracts:
  - Routing keys follow `<suite>.<domain>.<aggregate>.<event>`.
  - Event schemas are versioned; backward-compatible changes preferred.
  - Producers own event definitions; consumers validate and evolve adapters.
- Delivery semantics:
  - At-least-once delivery; consumers must be idempotent.
  - Durable queues and dead-letter strategy for poison messages.
  - Ordering is not guaranteed across aggregates; avoid relying on global order.

### 4.3 Pattern synergy and examples
- T1 synchronous lookups (dashed dependency) from higher tiers:
  - `pps.pd` → `t1.ref` for codes and labels; `t1.si` for units; `t1.i18n` for translations; `tech.dms`/`t1.rpt` for templates/PDF rendering.
- T2 cross-suite consumption (sync + async):
  - Any T3 domain → `shared.bp` for party lookup (sync REST) and `shared.bp.party.*` events for master-data sync (async).
  - Any T3 domain → `auto.wf` as an event producer that triggers automation; `auto.ntf` for multi-channel user notification.
- Cross-suite event flows (solid dependency):
  - `pps.pd.product.released` → drives `pps.mrp`, `pps.pur`, `pps.mes`, `pps.qm`, `pps.im`.
  - `pps.mrp.purchaseproposal.created` → informs `pps.pur`.
  - `pps.pur.purchaseorder.created` → sets expectations in `pps.im`; `pps.pur.asn.created` → `pps.wm`.
  - `pps.im.goodsreceipt.posted` → informs `pps.pur` and can trigger `pps.qm`.
  - `sd.sd.delivery.created` → consumed by `pps.im` for goods issue.
  - `sd.sd.billing.created` → consumed by `fi.ar` (financial posting).
  - `fi.ap.invoice.posted` → consumed by `pps.pur` (3-way match lifecycle).
  - `shared.bp.party.created` → consumed by `pps.pur`, `sd.sd`, `hr.hr`, `fi.ap`.
  - `auto.wf.workflow.triggered` → fan-out to REST actions across any suite.
  - Tier 4 `bi` consumes events from all suites for analytics.


## 5. Governance, Versioning, and Compatibility
- API versioning in path (`/v1`, `/v2`); additive changes are preferred. Breaking changes require a new major version.
- Event versioning via schema evolution and semantic version in headers/payload; support dual-publish during transitions.
- Backward compatibility guarantees documented per domain; sunset windows for deprecated fields.
- Contract-first development: OpenAPI for HTTP; AsyncAPI for event streams.
- Deprecated domains (see §2.3 CRM notes) publish under both legacy and new routing keys for a 60-day grace period before the bridge is removed.


## 6. Reliability and Operations
- Error handling: Clear error contracts; circuit breakers and timeouts for sync calls; DLQ monitoring for async.
- Idempotency: Required for event consumers and recommended for mutating HTTP calls.
- Observability: Structured logging with correlation IDs, metrics (latency, throughput, error rates), distributed tracing across sync and async hops.
- Security: AuthN/Z mediated by API gateway/service mesh; least-privilege for message broker access; data protection at rest and in transit.


## 7. Example Developer Checklist
- Before adding a new sync dependency, prefer T1 lookups or `shared.*` reads; avoid tight coupling to T3 services.
- Before adding an event, confirm it is a business fact, not a command; define a clear event name and schema.
- Before hand-rolling automation, check if `auto.wf` workflows + `auto.ntf` templates can cover the need declaratively.
- Ensure idempotent handlers and resilient consumers (retry, backoff, DLQ).
- Update topology diagrams in `dev.hub/architecture/system-topology.md` if topology changes.


## 8. References
- System topology & event-flow diagram: [`system-topology.md`](system-topology.md)
- Four-tier architecture overview: [`four-tier-architecture.md`](four-tier-architecture.md)
- Repository & naming strategy: [`../naming/repo-naming-strategy.md`](../naming/repo-naming-strategy.md)
- Tier specs: [`io.openleap.dev.spec`](https://github.com/openleap-io/io.openleap.dev.spec) → `T1_Platform`, `T2_Common`, `T3_Domains`, `T4_Data`
- Specification templates: [`io.openleap.dev.concepts`](https://github.com/openleap-io/io.openleap.dev.concepts) → `templates/platform/`
- Repo landscape & status: [`../landscape/`](../landscape/)
