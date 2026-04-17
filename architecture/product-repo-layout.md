# Product Repository Layout

This document defines the repository convention for **product specifications** in OpenLeap — the Application-Engineering counterpart to the platform-feature catalog held in `dev.spec`.

It is a conceptual proposal. It does not change any existing repo. It fills a documented gap: today's docs describe how products are *composed* in Elara but do not say *where the product spec is persisted for review, versioning, and release*.

---

## 1. Motivation

The SPLE model (see [sple-platform-concept.md](sple-platform-concept.md)) separates two lifecycles:

- **Platform Engineering** — reusable assets; specified in `dev.spec`, implemented in `{suite}.{domain}` services and `ui.*` libraries.
- **Application Engineering** — products derived for specific customers or market segments.

[integrated-overview.md §8.2](integrated-overview.md) states that *"products are assembled in Elara by selecting features from the catalog"* but stops short of defining where the resulting artifact lives outside Elara's database. For serious use this is insufficient: customers need reviewable PRs, reproducible builds, audit trails, air-gapped releases, and multi-variant branching.

The convention below introduces a Git-backed product-spec repository that complements Elara without replacing it. Elara remains the authoring UI; the repo is the release artifact.

---

## 2. The `prod` Namespace

A new category infix is added to the [repo naming strategy](../naming/repo-naming-strategy.md):

| Infix | Meaning | Examples |
|-------|---------|----------|
| `prod` | Product-specific artifacts (Application Engineering) | `prod.acme-sales`, `prod.acme-sales.ui`, `prod.acme-sales.bff` |

The three-repo product group:

```
io.openleap.prod.{product}           Product specification      (spec)
io.openleap.prod.{product}.ui        Product frontend           (Nuxt 4 / Vue 3)
io.openleap.prod.{product}.bff       Product BFF (optional)     (Spring Boot 4)
```

`{product}` is a short kebab-case identifier (`acme-sales`, `beta-service-portal`). It is not a suite — products cross suites freely.

Note: repos currently living at `io.openleap.{product}.ui` (e.g. `elara.ui`, `telos.ui`, `iam.ui`, `tech.dms.ui`) predate this convention. They are in-house platform tools rather than customer-derived products, so they are grandfathered under `ui.*` or their own namespace. New customer products adopt `prod.*`.

---

## 3. Contents of `io.openleap.prod.{product}`

```
io.openleap.prod.acme-sales/
├── README.md                      Persona map, process scope, release notes
├── product.yaml                   Product manifest (id, owner, target customer, version)
├── features/
│   └── selection.uvl              Feature selection: references F-{SUITE}-{NNN}
│                                  from dev.spec; never copies feature specs
├── variability/
│   └── bindings.yaml              Resolved UVL attributes (compile/deploy/runtime)
├── extensions/
│   ├── F-SD-001/                  Fills extension points declared by the
│   │   ├── fields.yaml            platform feature (see sple-platform-concept.md §6)
│   │   ├── rules.yaml
│   │   └── actions.yaml
│   └── F-PPS-012/
├── personas/
│   ├── sales-rep.yaml             UX-enriched persona (derived in Elara)
│   └── sales-manager.yaml
├── processes/
│   └── order-to-cash.bpmn         Customer process driving feature selection
├── screen-overrides/              Product-specific AUI overrides / CUI picks
│   └── F-SD-001-02.aui.yaml
├── bff/
│   └── productconfig.yaml         Runtime artifact consumed by the BFF:
│                                  active features, inclusion modes, extension routes
└── CHANGELOG.md
```

### 3.1 Reference, Do Not Copy

Entries in `features/selection.uvl` reference platform features by stable ID:

```uvl
features
    Product "Acme Sales"
        mandatory
            F-SD-001 "Create Sales Order"          // from dev.spec
            F-BP-001 "Business Partner"            // from dev.spec
            F-PPS-010 "Stock Read"                 // from dev.spec (read-only inclusion)
        optional
            F-FI-007 "Invoice Preview"

    constraints
        F-SD-001 requires F-BP-001
```

A product repo MUST NOT copy platform specs. It only points at them and extends them at declared extension points. The UVL validator (see [sple-platform-concept.md §4.3](sple-platform-concept.md)) resolves these references against `dev.spec`.

### 3.2 The `productconfig.yaml` Artifact

`bff/productconfig.yaml` is the release artifact consumed at runtime. It is generated from the rest of the repo by a packaging step and contains:

- Active features with inclusion mode (`full`, `read-only`, `embedded`, `excluded`)
- Resolved variability attribute values
- Extension-point routing (which custom endpoint or service handles each filled extension)
- BFF aggregation hints derived from Feature Spec §5 / §5.2

Per [sple-platform-concept.md §4.2](sple-platform-concept.md), the BFF reads this file to enforce feature-gating and compose view models.

---

## 4. Relationship Between the Three Product Repos

```
io.openleap.prod.acme-sales            (spec — source of truth, Git-reviewed)
        │
        │ productconfig.yaml  (release artifact)
        ▼
io.openleap.prod.acme-sales.bff        (runtime — reads productconfig.yaml,
        ▲                               calls platform services, enforces gating)
        │ BFF contract
        │
io.openleap.prod.acme-sales.ui         (frontend — calls the BFF, not the
                                        platform services directly)
```

The BFF repo is optional. Start co-located inside the UI repo; split only when multiple products share a non-trivial BFF codebase.

---

## 5. Elara's Role Under This Convention

Elara does not go away. It remains the **authoring UI** for the spec repo:

1. Product owner opens Elara, reads `dev.spec` catalog.
2. Selects features, binds variability, fills extensions, designs personas, imports processes.
3. Elara exports the result as a pull request against `io.openleap.prod.{product}`.
4. Platform and product architects review the PR; changes merge on approval.
5. CI packages `productconfig.yaml` and publishes a product release.

This preserves the Elara UX and adds Git discipline. Elara's database becomes a working area, not a system of record.

---

## 6. Alternative Layouts Considered

| Option | Shape | Rejected because |
|--------|-------|------------------|
| Central `dev.products` repo (all products in one) | `dev.products/acme-sales/`, `dev.products/beta/` | Couples unrelated customer release cadences; no per-product access control; scales badly past a handful. |
| Per-product folder inside `dev.spec` | `dev.spec/products/acme-sales/` | Blurs the platform/product boundary that `dev.spec` enforces; forces platform reviewers to review product selections. |
| No Git repo (status quo) | ProductConfig in Elara DB only | Fine for prototyping; no review, no reproducibility, no multi-customer branching. |
| Product spec inside `{product}.ui` | `{product}.ui/spec/` | Ties spec lifecycle to UI release; awkward when BFF or persona changes without UI changes. |

---

## 7. Multi-Customer Variants

Two patterns work, with different tradeoffs:

| Pattern | Shape | When to use |
|---------|-------|-------------|
| **One repo per customer deployment** | `prod.acme-sales`, `prod.beta-sales`, `prod.gamma-sales` | Early stage, few customers, divergent processes. Simple, but duplication accumulates. |
| **One repo per product, customers as overlays** | `prod.sales/` + `prod.sales/customers/acme/` branches or overlay folders | Mature stage, many customers, shared core. SPLE-orthodox but demands discipline. |

Start with the first pattern; migrate to the second when duplication becomes painful.

---

## 8. Related Edits and Follow-Ups

Adopting this convention requires aligned updates elsewhere. Status as of this document:

1. [naming/repo-naming-strategy.md](../naming/repo-naming-strategy.md) — `prod` infix added to the category table, mental-model, and namespace section. **Done.**
2. [sple-platform-concept.md §3.1](sple-platform-concept.md) — Product definition aligned with this repository convention. **Done.**
3. [integrated-overview.md §8.2](integrated-overview.md) and glossary — ProductConfig persistence in `io.openleap.prod.{product}` documented; Product glossary entry corrected. **Done.**
4. [landscape/repo-catalog.yaml](../landscape/repo-catalog.yaml) — `product-spec` and `product-bff` categories added to the header; reserved section added for future `prod.*` entries. **Done.**
5. `dev.concepts` — templates for `product.yaml`, `selection.uvl`, `bindings.yaml`, `extensions/*`, persona, process. **Pending in [`io.openleap.dev.concepts`](https://github.com/openleap-io/io.openleap.dev.concepts).** Out of scope for `dev.hub`: the spec-template system lives in that repo and must be extended there so the templates become discoverable alongside the existing `feature-spec`, `domain-service-spec`, etc.

---

## 9. Related Documents

- [sple-platform-concept.md](sple-platform-concept.md) — two-lifecycle model, platform-feature specification artifacts, extension points
- [integrated-overview.md](integrated-overview.md) — full architecture overview, Product Line Engineering section
- [four-tier-architecture.md](four-tier-architecture.md) — tier model and suite boundaries
- [naming/repo-naming-strategy.md](../naming/repo-naming-strategy.md) — repository naming conventions
