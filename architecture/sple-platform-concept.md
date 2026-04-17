# SPLE Microservice Platform Concept

OpenLeap is a **Software Product Line** (SPL) built on microservices. This document explains the foundational model: how Platform Engineering and Application Engineering work together, and what specification artifacts glue the two worlds together.

---

## 1. The Two-Lifecycle Model

Software Product Line Engineering (SPLE) separates the work of building reusable assets from the work of assembling specific products. OpenLeap follows the two-lifecycle model established by Pohl, Bockle & van der Linden (2005):

```
┌─────────────────────────────────────────────────────────────────┐
│                    PLATFORM ENGINEERING                          │
│                    (Domain Engineering)                          │
│                                                                 │
│  "What can the platform do?"                                    │
│                                                                 │
│  Build reusable assets: services, features, components,         │
│  contracts, screen specifications, extension points.            │
│  Declare variability. Design for many products, not one.        │
│                                                                 │
│  Tool: Telos (architecture & specification)                     │
│  Space: Solution Space                                          │
│  Repos: dev.spec, dev.concepts, fi.*, crm.*, sd.*, ...         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────┐      │
│  │            SPECIFICATION ARTIFACTS                     │      │
│  │      (the glue between the two worlds)                │      │
│  │                                                       │      │
│  │  Feature Specs + Feature Models (UVL) +               │      │
│  │  Screen Contracts (AUI/CUI) + BFF Contracts           │      │
│  └───────────────────────────────────────────────────────┘      │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                    APPLICATION ENGINEERING                       │
│                    (Product Derivation)                          │
│                                                                 │
│  "What does this customer need?"                                │
│                                                                 │
│  Select features from the platform catalog. Resolve             │
│  variability. Fill extension points. Compose products           │
│  for specific personas and business processes.                  │
│                                                                 │
│  Tool: Elara (discovery & configuration)                        │
│  Space: Problem Space / Product Space                           │
│  Repos: ui.admin, ui.elara, product-specific UIs                │
└─────────────────────────────────────────────────────────────────┘
```

This separation is not an implementation detail — it is the **fundamental architectural principle** of the platform. Every design decision flows from this divide.

---

## 2. Platform Engineering — Building Reusable Assets

Platform Engineering builds the catalog of reusable capabilities that products draw from. It organizes work across four tiers and several specification layers.

### 2.1 Four-Tier Service Architecture

Services are organized into tiers that govern dependency direction. Higher tiers depend on lower tiers, never the reverse.

```
T1 — Platform & Technical Foundations
     IAM, reference data, i18n, document management, job control
     Suite-agnostic. Sync lookups. Invisible to business users.

T2 — Shared Enterprise Business
     Business Partner, Calendar
     Cross-suite master data. DDD shared kernel.

T3 — Core Business Suites
     FI (Finance), CRM, SD (Sales), PS (Projects), SRV (Services), PPS (Production), ...
     Suite-specific bounded contexts. Domain events. This is where features live.

T4 — Data, Analytics & Integration
     BI, ETL pipelines, data lake
     Read-only consumer of all domain events. No features.
```

See [four-tier-architecture.md](four-tier-architecture.md) for the full description.

### 2.2 Suites, Domains, and Services

The platform uses Domain-Driven Design (Evans, 2003) to structure business logic:

| Concept | Definition | Example |
|---------|-----------|---------|
| **Suite** | A cluster of domains that share a common Ubiquitous Language (UBL). The UBL boundary IS the suite boundary. | PPS (Production): all domains understand "product", "BOM", "routing", "work order" identically |
| **Domain** | A bounded context within a suite. Owns aggregates, invariants, and domain events. | `pps.pd` (Product Definition), `pps.im` (Inventory Management) |
| **Service** | A deployable microservice implementing exactly one domain. Own database, own API, own event exchange. | `io.openleap.fi.coa` — Chart of Accounts service |

When a term crosses a suite boundary, its meaning changes and must be explicitly translated (Anti-Corruption Layer). "Document" in FI means a financial posting; in DMS it means a stored file.

### 2.3 The Platform Feature Catalog

The platform feature catalog is the heart of the product line. It contains all reusable capabilities, specified in a way that products can select, configure, and extend them.

Features are **owned by suites**, not products. They follow a hierarchy based on Feature-Oriented Domain Analysis (FODA, Kang et al., 1990):

```
Capability (composition node)
  "What can the system do in this area?"
  e.g., "Inventory Management"
    │
    ├── Use-Case (composition node or leaf)
    │   "What can the user accomplish?"
    │   e.g., "Post Goods Receipt"
    │     │
    │     └── Screen (leaf)
    │         "What does the user see?"
    │         e.g., "Goods Receipt Form"
    │
    └── Use-Case ...
```

Feature identity: `F-{SUITE}-{NNN}` (composition node) or `F-{SUITE}-{NNN}-{NN}` (leaf). The suite prefix indicates ownership.

---

## 3. Application Engineering — Deriving Products

Application Engineering takes the platform catalog and assembles specific products for customers or market segments.

### 3.1 Products

A **Product** is a composed application — spec, UI, and runtime BFF — that serves one or more personas. Products are independent of suites: a "Sales Workbench" product might include features from SD, BP, FI, and PPS.

Product composition is customer-driven: a product owner selects features from the platform catalog based on the customer's processes and personas. Authoring happens in Elara; the resulting specification is persisted as a Git repository (`io.openleap.prod.{product}`) so it can be reviewed, versioned, and released. See [product-repo-layout.md](product-repo-layout.md) for the full repository convention.

### 3.2 Product-Features

A **Product-Feature** is a configured instance of a platform-feature. It resolves variability and fills extension points for a specific product:

1. **Base:** Reference to the platform-feature (`F-PPS-012`)
2. **Configuration:** Variability resolved (attribute values set, optional sub-features decided)
3. **Extensions:** Extension points filled (customer-specific fields, rules, actions)

A product-feature may NOT modify the platform-feature's core logic, add functionality outside defined extension points, or change the backend dependency structure.

### 3.3 Personas and Processes

- **Personas** are UX-enriched user profiles derived from business actors discovered in Elara. They drive which features a product includes.
- **Processes** are customer-specific business workflows (BPMN). They inform feature selection: which features support the steps in this process?

---

## 4. The Specification Artifacts — Gluing the Two Worlds

The key challenge in any product line is: how do the two engineering lifecycles communicate? In OpenLeap, the answer is **specification artifacts** — formal documents that are authored in Platform Engineering and consumed in Application Engineering.

### 4.1 The Platform-Feature Specification (The Central Artifact)

A platform-feature is not one document but **four integrated artifacts** managed together:

```
Platform-Feature F-{SUITE}-{NNN}
├── feature-spec.md        — What the feature does (human-readable)
├── feature-model.uvl      — How the feature varies (machine-readable)
├── screen-contract.aui    — What the user sees, platform-free (YAML)
└── screen-contract.cui    — Platform-specific rendering (per target)
```

These are **four views of the same feature**, not independent deliverables:

| Artifact | Format | Authored by | Consumed by | What it specifies |
|----------|--------|------------|-------------|-------------------|
| **Feature Spec** | Markdown | Platform Engineering | Both | User goals, scenarios, journey, interaction requirements, backend dependencies, acceptance criteria, extension points |
| **Feature Model** | UVL | Platform Engineering | Application Engineering | Position in feature tree, variability (mandatory/optional/alternative), attributes with binding times, cross-tree constraints |
| **Screen Contract (AUI)** | YAML | Platform Engineering | Application Engineering | Task model, layout zones, absent-rules, business rules, data bindings — platform-free |
| **Screen Contract (CUI)** | YAML | Platform Engineering | Product UI teams | Platform-specific rendering (web: responsive breakpoints, mobile: touch, desktop: keyboard-first) |

The AUI/CUI split follows the Cameleon Reference Framework (Calvary et al., 2003): one abstract specification, multiple concrete renderings.

### 4.2 The BFF Contract (The Runtime Bridge)

The **Backend for Frontend (BFF)** is the runtime layer that connects product-specific frontends to platform-agnostic backend services. Its behavior is not free-form — it is specified by the feature spec:

```
Feature Spec section 5 (Backend Dependencies)
├── Service calls: which domain services, which endpoints, which tier
├── Aggregation rules: how responses are merged into a view model
├── Failure modes: what happens if a service is unavailable
└── isMutation flag: read vs. write (affects feature-gating)

Feature Spec section 5.2 (View-Model Shape)
└── JSON example of what the BFF returns to the frontend
    This IS the BFF response contract.
```

At runtime, a BFF instance is configured by the **ProductConfig** from Elara:

- Active features and their inclusion mode (`full`, `read-only`, `excluded`)
- Resolved variability attributes
- Extension point mappings (which custom services handle which extensions)

The BFF enforces feature-gating: if a feature is `read-only`, write endpoints are blocked. If a feature is `excluded`, its backend calls are not made at all. The frontend does not need to know about platform-level configuration — the BFF handles it.

### 4.3 The UVL Feature Model (The Variability Contract)

Feature models are expressed in UVL (Universal Variability Language, Sundermann et al., 2021). They are machine-readable and enable automated validation:

```uvl
features
    F-SD-001 "Create Sales Order"
        mandatory
            F-SD-001-01 "Order Overview"
            F-SD-001-02 "Order Entry"
        optional
            F-SD-001-03 "Order Analytics"

        constraints
            F-SD-001 requires F-PPS-010   // needs stock data (cross-suite read)
            F-SD-001 requires F-BP-001    // needs customer data (cross-suite read)
```

During product configuration, the UVL validator checks that all `requires` constraints are satisfied. A product that includes `F-SD-001` but excludes `F-BP-001` is rejected — the configuration is incomplete.

### 4.4 How the Artifacts Flow Between the Two Worlds

```
PLATFORM ENGINEERING                         APPLICATION ENGINEERING
(build reusable assets)                      (derive specific products)

Feature Spec (.md)  ─────────────────────►  Product owner reads to understand
                                             what the feature does

Feature Model (.uvl) ────────────────────►  Elara uses to validate product
                                             configuration completeness

Screen Contract (AUI) ──────────────────►  Product UI team uses as the
                                             stable interface to implement against

Screen Contract (CUI) ──────────────────►  Framework-specific implementation
                                             guide (web, mobile, desktop)

BFF Contract (section 5 + 5.2) ──────────►  BFF configured by ProductConfig
                                             to compose view models, enforce
                                             feature-gating, route extensions

Extension Points ────────────────────────►  Product team fills with
                                             customer-specific logic
```

### 4.5 The Feature Lifecycle Across Both Worlds

| Phase | Engineering | Activity | Artifact produced |
|-------|------------|----------|-------------------|
| Discovery | Application | AI-guided interviews identify business needs | Feature Candidate (in Elara) |
| Cataloging | Platform | Feature specified and added to catalog | Feature Spec + UVL + AUI + CUI(s) |
| Selection | Application | Product owner selects features for a product | ProductConfig |
| Configuration | Application | Variability resolved, attributes set | Configured Product-Feature |
| Extension | Application | Extension points filled with custom logic | Extended Product-Feature |
| Implementation | Both | Backend services + BFF + UI built | Running system |

---

## 5. Cross-Suite Interaction Rules

Features may span multiple suites (e.g., "Create Sales Order" reads stock data from PPS). The platform enforces strict rules:

| Rule | Description |
|------|-------------|
| **Read-across, mutate-local** | A feature may read data from any suite but may only write to services within its own suite |
| **Cross-suite mutations via events** | When suite A needs to trigger a change in suite B, it publishes a domain event. Suite B reacts asynchronously (eventual consistency) |
| **Every cross-suite read creates a dependency** | If `F-SD-001` reads from PPS, it must declare `requires F-PPS-010` in its UVL model |
| **Product validation enforces completeness** | The UVL validator rejects products with missing required features |

---

## 6. Extension Points — The Open-Closed Principle

Platform assets are open for extension but closed for modification (Meyer, 1988). Extension points are declared in the platform, not invented by products:

| Level | Extension type | Example |
|-------|---------------|---------|
| Feature | Extension zones, fields, rules, actions in screen contracts | "Add custom fields to the Goods Receipt form" |
| Domain | Extension events, aggregate hooks | "React to custom events on the Product aggregate" |
| Service | Extension API endpoints, message handlers | "Add a custom endpoint to the PPS-PD service" |

Products extend at defined points. They do not fork or modify platform code.

---

## 7. Scientific and Industry Foundations

| Concept | Source | How OpenLeap uses it |
|---------|--------|---------------------|
| Software Product Lines | Pohl, Bockle & van der Linden (2005) | Two-lifecycle model: Domain Engineering / Application Engineering |
| Feature-Oriented Domain Analysis | Kang et al. (1990), SEI/CMU | Feature hierarchy as the central variability mechanism |
| Universal Variability Language | Sundermann et al. (2021), MODEVAR community | Machine-readable feature models with attributes and constraints |
| Cameleon Reference Framework | Calvary et al. (2003) | AUI/CUI split for multi-platform UI derivation |
| Domain-Driven Design | Evans (2003) | Bounded contexts, ubiquitous language, aggregates |
| CQRS + Event patterns | Young (2010), Fowler (2011) | Command/Query separation, thin domain events, outbox pattern |
| Microservice architecture | Newman (2015) | One service per bounded context, independent deployability |
| Open-Closed Principle | Meyer (1988) | Extension points: open for extension, closed for modification |
| Generative Programming | Czarnecki & Eisenecker (2000) | Feature as the central mediating artifact between platform and product |

---

## 8. Summary

```
Platform Engineering          Specification Artifacts          Application Engineering
(build for many)              (the contracts)                  (build for one)
                                                               
Suites & Domains         ──►  Feature Specs            ──►    Products & Personas
Services & APIs          ──►  Feature Models (UVL)     ──►    Product Configuration
Screen Contracts (AUI)   ──►  BFF Contracts            ──►    Product UI + BFF
Extension Points         ──►  Extension Declarations   ──►    Customer Extensions

                    ┌──────────────────────────┐
                    │  Feature = Central Glue   │
                    │                          │
                    │  Spec + UVL + AUI + BFF  │
                    │  = one feature, 4 views  │
                    └──────────────────────────┘
```

The feature is the **central mediating artifact** (Czarnecki & Eisenecker, 2000) between Platform Engineering and Application Engineering. It is specified in the platform, selected and configured in the product, and executed at runtime through the BFF. All four views (spec, UVL, AUI, BFF contract) must stay in sync — they are not independent artifacts but four facets of the same capability.

---

## Related Documents

- [integrated-overview.md](integrated-overview.md) — Full architecture overview including data architecture, communication patterns, and AI tooling
- [four-tier-architecture.md](four-tier-architecture.md) — Detailed tier descriptions with suite and domain listings
- [system-topology.md](system-topology.md) — System layout with Mermaid event flow diagrams
- [dev.concepts](https://github.com/openleap-io/io.openleap.dev.concepts) — The full conceptual stack, artifact catalog, templates, and governance
