# T4 — Data, Analytics & Data Activation

This document revises and replaces the one-line T4 definition in [four-tier-architecture.md §2.4](four-tier-architecture.md). It is based on a state-of-the-art review of modern data architectures (medallion, lakehouse, data mesh) and ERP precedents (SAP BTP / Datasphere / BW/4HANA, Oracle Fusion Analytics, Microsoft Fabric), together with open standards published by the Linux Foundation (ODCS, ODPS) and the Apache Software Foundation (Iceberg, Polaris).

---

## 1. Where T4 Sits

T4 is the **data and analytics plane** of the OpenLeap platform. It consumes domain events from all T3 suites (and selectively from T1/T2), curates them into governed data products, and serves analytical, ML, and activation workloads.

```
┌──────────────────────────────────────────────────────────┐
│  T1 — Platform & Technical Foundations                    │
│  T2 — Shared Enterprise Business                          │
│  T3 — Core Business Suites   ─── domain events ──┐       │
├──────────────────────────────────────────────────┼───────┤
│  T4 — Data, Analytics & Data Activation          ▼       │
│  ├── Data Platform (lake, stream, warehouse, …)          │
│  ├── Analytics & AI (metrics, BI, MLOps, …)              │
│  └── Data Activation (reverse-ETL, data products)        │
└──────────────────────────────────────────────────────────┘
          │
          │ read-only back-pressure via public T3 APIs
          ▼  (never direct writes to T3 state)
      T3 suites
```

### Invariants

| Rule | Statement |
|------|-----------|
| **Read-only of T3 state** | T4 consumes T3 domain events. It never writes directly to T3 databases. Any write-back goes through T3 public APIs via `t4.reverse`. Inherited from [sple-platform-concept.md §5](sple-platform-concept.md). |
| **T4 may emit its own events** | T4 publishes *analytical* events (KPI breaches, ML predictions, drift alerts) on `t4.{domain}.events` exchanges. These are **not** domain events in the DDD sense — they are analytical notifications. The older rule "T4 never produces events" is rescinded; the correct rule is "T4 never produces domain events." |
| **Suite-owned data products** | Curated datasets published in `t4.warehouse` are *owned by the producing T3 suite*, not by T4. T4 provides the platform (pipelines, governance, catalog, SLAs); the suite owns the contract. This follows Dehghani's data mesh principles.[^mesh-principles] |
| **No forking of platform dashboards/metrics** | Products consume platform dashboards and metrics through selection + binding, like any other SPLE feature. Products may extend at declared extension points; they may not modify template internals. |
| **Data contracts are first-class SPLE artifacts** | Data products are specified via ODCS (Open Data Contract Standard)[^odcs] and ODPS (Open Data Product Specification)[^odps]. These sit alongside feature specs, UVL models, and screen contracts as inputs to product composition. |

---

## 2. T4 Is Both a Platform and an Application Concern

The SPLE two-lifecycle model ([sple-platform-concept.md §1](sple-platform-concept.md)) applies **recursively** inside T4. The platform/product split is not a property of T3 alone.

| Side | What lives here | Repo |
|------|-----------------|------|
| **T4 Platform Engineering** | Data infrastructure (lake, stream, warehouse, catalog, quality, governance, privacy, MDM, sharing). Canonical data products per suite. Dashboard templates, metric DSL, connector SDK, model-serving runtime. These ARE platform features, carrying UVL variability, extension points, and BFF contracts. | `dev.spec/T4/*`, `t4.*` services |
| **T4 Application Engineering** | Product-specific dashboards (selections from template catalog), custom KPI definitions bound to the DSL, tenant/customer-specific ML models, product-specific data activations. | `io.openleap.prod.{product}/dashboards/`, `metrics/`, `integrations/`, `models/` |

An earlier draft of this document claimed "T4 has no features." That was wrong. T4 features exist — they just look different from T3 features (a dashboard template is a feature; a metric definition is a feature; a data product is a feature).

The product-repo layout ([product-repo-layout.md §3](product-repo-layout.md)) must accommodate these T4 artifacts. This is tracked as a follow-up in §6 below.

---

## 3. Three Pillars

T4 is structured as three pillars, aligned with the SAP Datasphere / Integration Suite split,[^sap-integration-vs-data] with Microsoft Fabric's lakehouse + real-time + analytics layering,[^fabric-medallion] and with Gartner's data fabric reference.[^gartner-fabric]

```
                           ┌────────────────────────────┐
 Pillar 1:                 │  DATA PLATFORM             │
 Foundation                │  "store, govern, share"    │
                           └─────────────┬──────────────┘
                                         │ data products
                           ┌─────────────▼──────────────┐
 Pillar 2:                 │  ANALYTICS & AI            │
 Consumption               │  "model, measure, predict" │
                           └─────────────┬──────────────┘
                                         │ insights, models
                           ┌─────────────▼──────────────┐
 Pillar 3:                 │  DATA ACTIVATION           │
 Outbound use              │  "operationalize, share"   │
                           └────────────────────────────┘
```

The third pillar is named **Data Activation**, not "Integration." Process integration (iPaaS, EDI, B2B) is an edge concern and does **not** belong in T4 — see §4.

### 3.1 Pillar 1 — Data Platform

The foundation: ingest events, store canonical data products, govern, share.

| Domain | Purpose | Notes |
|--------|---------|-------|
| **`t4.lake`** | Raw event ingestion and object storage. Open table formats (Apache Iceberg[^iceberg] / Delta / Hudi). | Medallion bronze/silver layers.[^databricks-medallion] |
| **`t4.stream`** | Real-time / streaming plane. Kafka-style durable log + stream processing (Flink, Materialize). Peer to `t4.lake`, not a sublayer.[^waehner-streaming-lakehouse] [^fabric-rti-medallion] | For an event-sourced ERP on RabbitMQ this is unavoidable. |
| **`t4.warehouse`** | Curated, modeled, governed datasets (medallion gold). Star / snowflake / vault models per suite. | Suite-owned, T4-platform-operated. |
| **`t4.catalog`** | Technical metadata, lineage, data-product registry. Open-catalog layer (Unity Catalog OSS,[^unity-oss] Apache Polaris[^polaris]). | Distinct from governance and from data contracts. |
| **`t4.contracts`** | Data-product contract lifecycle. Authoring + versioning + validation of ODCS[^odcs] and ODPS[^odps] documents. | Data-plane analogue of `dev.spec`. First-class SPLE artifact. |
| **`t4.quality`** | Data-quality rules, SLAs, SLOs, freshness monitors, anomaly detection. | Per-contract enforcement; feeds `t4.governance` alerts. |
| **`t4.governance`** | Federated computational governance per Dehghani's 4th principle:[^mesh-principles] policy-as-code, classification, access control. | Distinct from `t4.catalog`. Consumes catalog metadata.[^privacera-governance] |
| **`t4.privacy`** | PII detection, classification, consent management, purpose binding. Driven by 2025 regulatory landscape (US state privacy laws, GDPR).[^privacera-governance] | Feeds `t4.governance` decisions. |
| **`t4.mdm`** | Master Data Management: golden-record mediation across suites where BP / Calendar (T2) is insufficient.[^informatica-mdm] | Not a T3 business domain; not a warehouse schema. Own domain in T4. |
| **`t4.sharing`** | Zero-copy data sharing (Delta Sharing, Snowflake Sharing, Unity/Polaris sharing).[^unity-oss] [^snowflake-sharing] [^databricks-sharing-2025] | Outbound but inside the data plane, not process integration. |

### 3.2 Pillar 2 — Analytics & AI

Consume data products; produce measurements, insights, predictions.

| Domain | Purpose | Notes |
|--------|---------|-------|
| **`t4.metrics`** | Headless semantic / metric layer.[^dbt-sl] [^cube-dbt] Metric DSL, definition registry, warehouse-agnostic query. | Critical for AI-native text-to-SQL accuracy.[^semantic-text-to-sql] Peer to `t4.warehouse`. |
| **`t4.bi`** | Dashboards, pixel-perfect reports, embedded analytics. Templates are platform features; instances are product-level selections. | The original "bi" placeholder lives here. |
| **`t4.explore`** | Ad-hoc SQL, notebook, self-service exploration. | Governed by `t4.governance` and `t4.privacy`. |
| **`t4.features`** | Feature store (offline + online) for ML. | Was implicit in the earlier "t4.ml"; split out per MLOps 2025 consensus.[^databricks-mlops] [^introl-mlops] |
| **`t4.mlops`** | ML lifecycle: reproducible pipelines, training, model registry, drift + bias monitoring, evaluation. | Separate from serving. |
| **`t4.serving`** | Model serving / inference. Replaces the old ambiguous `t4.ml`. | |
| **`t4.agents`** | *(optional, provisional)* Agent-driven analytics bridge to Agora. Text-to-SQL, narrative insights, decision agents. | Boundary with Agora (Elara / Telos / Noa) to be resolved — see §6. |

### 3.3 Pillar 3 — Data Activation

Push curated data out — inside or outside the platform — without blurring into process integration.

| Domain | Purpose | Notes |
|--------|---------|-------|
| **`t4.reverse`** | Reverse-ETL: push curated data from the warehouse back to T3 operational services via public APIs.[^reverse-etl-hightouch] [^reverse-etl-fivetran] | Only data-plane write-back path; still never writes T3 state directly. |
| **`t4.products`** | Internal data-product marketplace per ODPS 4.1.[^odps] The "catalog of consumable data products" — analogue of the feature catalog. | Consumers: product owners authoring `prod.{product}`, external partners via `t4.sharing`. |

---

## 4. What's NOT in T4 — Integration Decomposed Across the Existing Tiers

Process integration is not a data-plane concern. SAP itself separates **Datasphere** (data fabric) from **Integration Suite** (iPaaS / process integration).[^sap-integration-vs-data] Oracle separates OIC from Fusion Analytics Warehouse. OpenLeap adopts the same split.

But integration is not *one* concern — it is **three layers**, each with a different home. Trying to put "integration" into any single tier fails because the layers have different semantic weights: the iPaaS runtime has no business semantics (belongs in a technical tier), while a NetSuite Lead connector is full of business semantics (belongs in the suite whose domain it touches). And the customer-specific binding belongs in the product.

| Layer | Example | Has business semantics? | Home |
|-------|---------|-------------------------|------|
| **Integration infrastructure** | iPaaS flow engine, retry / DLQ, circuit breaker, observability; B2B gateway middleware; EDI *format* parsers (EDIFACT tokenizer, X12 segment reader); AS2 / SFTP / OAuth plumbing; credential vault | No — technical plumbing only | **T1** (alongside `ops.gateway`, `ops.config`, `ops.registry`) |
| **Semantic connectors** | `NetSuiteLeadSyncConnector` references `Lead`; `SAPInvoiceEDIMapper` references `Invoice`; `HubSpotContactConnector` references `Contact` | Yes — suite-owned business semantics | **T3 suites** — proposed sub-namespace `crm.int.netsuite`, `fi.int.sap-edi`, etc. |
| **Integration binding** | "`prod.acme-sales` uses `NetSuiteLeadSyncConnector` v3, OAuth app `acme-nso`, maps `Lead.source-campaign-id` ↔ NetSuite custom field `cf_camp_id`, triggers on `lead.qualified`" | Customer-specific | **`io.openleap.prod.{product}/integrations/`** |

This is the same decomposition as data in T4: T1 provides technical plumbing, suites own the semantic pieces, products instantiate them. It is the same decomposition as dashboards: T4 provides templates, products configure instances.

```
Technical plumbing              Semantic connector              Product binding
(no business semantics)         (suite-owned)                   (customer-specific)
─────────────────────           ──────────────────────          ───────────────────────
T1 iPaaS engine             →   crm.int.netsuite            →   prod.acme-sales/
T1 EDI parser (format)          (LeadSyncConnector)             integrations/netsuite.yaml
T1 B2B gateway                  fi.int.sap-edi              →   prod.acme-sales/
                                (InvoiceEDIMapper)              integrations/sap-edi.yaml
```

The `tech.zugferd` precedent fits this model cleanly: ZUGFeRD is a *format*, therefore technical, therefore T1. A hypothetical `fi.int.zugferd-invoice` that mapped ZUGFeRD messages to FI invoice concepts would live in the FI suite (semantic).

### What moves out of T4

Given the decomposition above, the old `t4.ipaas` / `t4.edi` / `t4.b2b` candidates split:

| Earlier label | Splits into | Homes |
|---------------|-------------|-------|
| `t4.ipaas` | iPaaS runtime (technical) + semantic connectors (per suite) + product bindings | T1 + T3 suites + `prod.*` |
| `t4.edi` | EDI format parser (technical) + EDI semantic mappers (per suite, e.g. `fi.int.edi-invoic`, `sd.int.edi-orders`) + product bindings | T1 + T3 suites + `prod.*` |
| `t4.b2b` | B2B gateway middleware (technical) + partner-specific semantic adapters (per suite if partner-data-shaped, or platform-level if protocol-only) + product bindings | T1 + T3 suites + `prod.*` |

### Why `t4.reverse` stays in T4

Reverse-ETL pushes *curated data* from `t4.warehouse` back to T3 services via their public APIs. It never leaves the OpenLeap boundary. That is a data-plane concern inside the platform; the iPaaS/EDI/B2B concern is about talking to *external* systems, which is a different problem with different protocols, security, and contract semantics — which is why SAP keeps Integration Suite and Datasphere separate.

### Open design questions

1. Is a dedicated `int.*` sub-namespace in each T3 suite the right shape (`crm.int.netsuite`), or should integration connectors be a sibling namespace (`int.crm.netsuite`), or a cross-cutting domain (`shared.int.*`) for partners that span suites? Decide before scaffolding.
2. Does the iPaaS runtime deserve its own T1 category distinct from `ops.*` (e.g. `int.runtime`, `t1.int`), or does it nestle under `ops.gateway` as an extension? A standalone category is cleaner once more than one integration engine is hosted.
3. For partners whose data spans suites (a customer's single SAP instance exports Orders, Invoices, *and* Master Data), does the semantic connector belong in one suite, split across suites, or in a T2-shared location? Likely split, but the multi-suite case deserves a canonical pattern.

---

## 5. SPLE Implications

Everything in [sple-platform-concept.md](sple-platform-concept.md) applies to T4. The feature lifecycle, promote-vs-extend decision, BFF contract, and cross-suite rules are not T3-specific.

| Concern | T3 example | T4 equivalent |
|---------|-----------|---------------|
| Platform feature | `F-CRM-001 "Manage Leads"` | `F-T4-BI-001 "Pipeline by Stage"` (dashboard template) |
| Variability | Lead-scoring model attribute | Metric grouping dimensions, filter defaults |
| Extension point | Custom fields on Lead | Custom columns, custom filters, derived metrics |
| UVL reference in product | `features/selection.uvl` | Same file; T4 features referenced alongside T3 |
| Product artifact | `extensions/F-CRM-001/fields.yaml` | `dashboards/sales-pipeline.yaml`, `metrics/opp-win-rate.yaml` |
| Gap handling | Promote new feature or extend | Identical — promote new dashboard template or extend an existing one |

Because T4 produces feature-like artifacts (dashboards, metrics, data-product contracts, connectors), these must become first-class template types in `dev.concepts`, the same way `feature-spec` and `domain-service-spec` are.

---

## 6. Follow-Ups

Deferred work triggered by this structure, with the repo that owns it:

1. **Integration decomposition design doc** — `dev.hub/architecture/` (new doc, e.g. `integration-decomposition.md`). Must precede any integration scaffolding. Resolves the three open design questions in §4 (namespace shape, iPaaS runtime placement, multi-suite partners) and specifies the T1 + T3-suite + `prod.*` split.
2. **Extend `product-repo-layout.md §3`** — add `dashboards/`, `metrics/`, `integrations/`, `models/` folders to `io.openleap.prod.{product}` and specify their content.
3. **Extend `getting-started-product.md`** — add a T4 slice to CRM Workbench (pipeline dashboard, invoice-aging KPI, NetSuite integration binding) and Ticket Desk (SLA dashboard, KB search analytics) worked examples.
4. **Agora / `t4.agents` boundary** — decide whether agent-driven analytics lives in Agora (existing), in `t4.agents`, or spans both with a clean contract.
5. **`dev.concepts` templates for T4 artifacts** — dashboard-spec, metric-spec, data-product-contract (ODCS), data-product (ODPS), connector-spec. Pending in [`io.openleap.dev.concepts`](https://github.com/openleap-io/io.openleap.dev.concepts).
6. **`dev.spec/T4_Data/`** — currently near-empty (1 file per `naming/repo-naming-strategy.md`). Needs scaffolding per the pillar/domain structure above. Pending in `dev.spec`.
7. **Rewrite `four-tier-architecture.md §2.4`** — current text lists only "bi — Data Platform" and is superseded by this document.
8. **`tech.zugferd` review** — confirm it stays in T1 as a *format* library (consistent with the decomposition in §4), while any *semantic* ZUGFeRD→FI-invoice mapper lives in `fi.int.*`. Trivially resolvable once §6.1 is done.

---

## 7. Citations

Primary sources cited inline above.

[^mesh-principles]: Zhamak Dehghani, *Data Mesh Principles and Logical Architecture*, martinfowler.com. https://martinfowler.com/articles/data-mesh-principles.html
[^databricks-medallion]: Databricks, *Medallion Lakehouse Architecture*. https://docs.databricks.com/aws/en/lakehouse/medallion
[^fabric-medallion]: Microsoft Learn, *Medallion Architecture in Fabric / OneLake*. https://learn.microsoft.com/en-us/fabric/onelake/onelake-medallion-lakehouse-architecture
[^fabric-rti-medallion]: Microsoft Learn, *Medallion in Real-Time Intelligence (Fabric)*. https://learn.microsoft.com/en-us/fabric/real-time-intelligence/architecture-medallion
[^gartner-fabric]: Gartner, *What is Data Fabric*. https://www.gartner.com/en/data-analytics/topics/data-fabric
[^sap-integration-vs-data]: SAP, *When to Use SAP Integration Suite vs SAP Datasphere*. https://help.sap.com/docs/sap-btp-guidance-framework/integration-architecture-guide/when-to-use-sap-integration-suite-vs-sap-datasphere
[^iceberg]: Apache Iceberg. https://iceberg.apache.org/
[^waehner-streaming-lakehouse]: Kai Waehner, *Data Streaming Meets Lakehouse — Apache Iceberg for Unified Real-Time and Batch Analytics*, Nov 2025. https://www.kai-waehner.de/blog/2025/11/19/data-streaming-meets-lakehouse-apache-iceberg-for-unified-real-time-and-batch-analytics/
[^unity-oss]: VentureBeat, *Databricks Open-Sources Unity Catalog*. https://venturebeat.com/data-infrastructure/databricks-open-sources-unity-catalog-challenging-snowflake-on-interoperability-for-data-workloads
[^polaris]: Apache Polaris (REST Iceberg catalog, ASF incubating). https://yeedu.com/posts/apache-polaris-data-catalog-for-open-data-platforms
[^odcs]: Bitol / Linux Foundation, *Open Data Contract Standard (ODCS) v3.1*. https://bitol-io.github.io/open-data-contract-standard/v3.1.0/
[^odps]: Linux Foundation, *Open Data Product Specification (ODPS) v4.1*, Oct 2025. https://opendataproducts.org/v4.1/
[^privacera-governance]: Privacera, *Unifying Data Catalogs and Security Governance*. https://privacera.com/blog/unifying-data-catalogs-and-security-governance-the-key-to-seamless-secure-and-auditable-data-access-privacera/
[^informatica-mdm]: Informatica, *MDM Integration Architecture*. https://www.informatica.com/resources/articles/mdm-integration-architecture.html
[^snowflake-sharing]: Snowflake, *Extending Data Sharing to Open Table Formats (Zero-ETL)*. https://www.snowflake.com/en/blog/data-sharing-open-table-formats/
[^databricks-sharing-2025]: Databricks, *What's New in Data Sharing — Summer 2025*. https://www.databricks.com/blog/whats-new-data-sharing-and-collaboration-summer-2025
[^dbt-sl]: dbt Labs, *dbt Semantic Layer*. https://docs.getdbt.com/docs/use-dbt-semantic-layer/dbt-sl
[^cube-dbt]: Cube, *dbt Metrics + Cube Integration*. https://cube.dev/blog/dbt-metrics-meet-cube
[^semantic-text-to-sql]: VentureBeat, *Headless vs Native Semantic Layer & Text-to-SQL Accuracy*. https://venturebeat.com/ai/headless-vs-native-semantic-layer-the-architectural-key-to-unlocking-90-text
[^databricks-mlops]: Databricks, *What is MLOps*. https://www.databricks.com/blog/what-is-mlops
[^introl-mlops]: Introl, *Model Registry and Governance 2025*. https://introl.com/blog/model-registry-governance-mlops-production-ai-2025
[^reverse-etl-hightouch]: Hightouch, *What is Reverse ETL*. https://hightouch.com/blog/reverse-etl
[^reverse-etl-fivetran]: Fivetran, *What is Reverse ETL*. https://www.fivetran.com/blog/what-is-reverse-etl

---

## 8. Related Documents

- [sple-platform-concept.md](sple-platform-concept.md) — two-lifecycle model, feature specification artifacts, cross-suite rules, extension points
- [four-tier-architecture.md](four-tier-architecture.md) — the tier model (§2.4 is superseded by this document)
- [product-repo-layout.md](product-repo-layout.md) — `prod.{product}` repo shape (to be extended per §6.2 above)
- [getting-started-product.md](getting-started-product.md) — end-to-end walkthrough (T4 slice to be added per §6.3 above)
- [integrated-overview.md](integrated-overview.md) — full architecture overview
- [naming/repo-naming-strategy.md](../naming/repo-naming-strategy.md) — naming conventions
