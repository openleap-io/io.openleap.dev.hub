# Repository Naming Strategy

## Overview

The openleap-io GitHub organisation has **128 repos**. This document defines a consistent naming convention for all repositories, with a focus on the non-domain "management/organisational" repos that currently lack structure.

---

## Naming Pattern

All repos follow the pattern:

```
io.openleap.{category}.{name}
```

Where `{category}` is either a **business domain** (for services) or an **infrastructure category** (for everything else).

---

## Categories

### Business Domain Services

Domain services use short abbreviations as their category infix. These are already well-established:

| Category | Domain | Examples |
|----------|--------|----------|
| `fi` | Finance | fi.coa, fi.gl, fi.ap, fi.ar, fi.bank |
| `crm` | CRM | crm.lead, crm.contact, crm.opp (note: `crm.ntf`, `crm.wf`, `crm.search`, `crm.email`, `crm.sup` are DEPRECATED and being promoted / superseded — see dev.spec) |
| `tks` | Ticket System | tks.tkt, tks.ch, tks.kb, tks.cmdb |
| `iam` | Identity & Access | iam.authz, iam.tenant, iam.principal, iam.audit |
| `ps` | Project Services | ps.prj, ps.bud, ps.tim |
| `sd` | Sales & Distribution | sd.ord, sd.bil, sd.shp |
| `srv` | Service Delivery | srv.cas, srv.apt, srv.cat |
| `pur` | Purchasing | pur.sup |
| `shared` | T2 suite — cross-domain master data | shared.bp, shared.cap |
| `auto` | T2 suite — cross-domain automation fabric | auto.ntf, auto.wf *(promoted from crm.ntf, crm.wf)* |
| `t1` | T1 — Platform params/ref services | t1.cfg, t1.i18n, t1.jc, t1.ref |
| `tech` | T1 — Technical services | tech.dms, tech.zugferd, tech.search, tech.email, tech.ai |
| `event` | Event/CDC | event.cdc |
| `common` | Shared Java libraries *(not a domain suite)* | common.data, common.nfs |

### Infrastructure Categories

Non-domain repos use one of four category infixes:

| Category | Meaning | What belongs here |
|----------|---------|-------------------|
| **`dev`** | Development | Standards, specifications, guidelines, templates, build tooling, shared libraries, test tooling |
| **`ops`** | Operations | Infrastructure-as-code, Kubernetes, CI/CD workflows, platform runtime services (gateway, config, registry) |
| **`ui`** | UI | Shared UI libraries, design system, UI guidelines, standalone UI applications |
| **`ai`** | AI | AI and automation tooling |
| **`prod`** | Product (Application Engineering) | Customer- or market-specific product specs, product UIs, product BFFs. See [architecture/product-repo-layout.md](../architecture/product-repo-layout.md). |

#### Mental model

```
io.openleap.dev.*       — build it
io.openleap.ops.*       — run it
io.openleap.ui.*        — show it
io.openleap.ai.*        — automate it
io.openleap.prod.*      — ship it (a product derived from the platform)
io.openleap.{domain}.*  — the business logic
```

---

## Rename Mapping

### `dev` — Developer Experience

| Current | New | Notes |
|---------|-----|-------|
| `io.openleap.spec` | **Split into 4 repos** | See [Spec Repo Split](#spec-repo-split) below |
| `io.openleap.mvp` | `io.openleap.dev.mvp` | MVP specifications |
| `io.openleap.dev.guidelines` | — | Already correct |
| `io.openleap.parent` | `io.openleap.dev.parent` | Parent POM / Maven build infrastructure |
| `io.openleap.starter` | `io.openleap.dev.starter` | Spring Boot starter library |
| `io.openleap.template` | `io.openleap.dev.template` | Microservice scaffold |
| `io.openleap.service-archetype` | `io.openleap.dev.archetype` | Maven archetype for new services |
| `io.openleap.bruno-collections` | `io.openleap.dev.bruno` | API testing collections |
| `io.openleap.tools.customize` | `io.openleap.dev.customize` | Customization tooling |
| `io.openleap.domain` | `io.openleap.dev.domain` | Shared domain library |
| `io.openleap.core` | `io.openleap.dev.core` | Core library |
| `io.openleap.jump-lib` | `io.openleap.dev.jump-lib` | Utility/helper library |

### `ops` — Operations & Infrastructure

| Current | New | Notes |
|---------|-----|-------|
| `io.openleap.iac` | `io.openleap.ops.iac` | Docker Compose / IaC |
| `io.openleap.k8s` | `io.openleap.ops.k8s` | Kubernetes manifests |
| `io.openleap.workflows` | `io.openleap.ops.workflows` | Reusable GitHub Actions workflows |
| `io.openleap.config` | `io.openleap.ops.config` | Spring Cloud Config Server |
| `io.openleap.config-repo` | `io.openleap.ops.config-repo` | Static configuration files |
| `io.openleap.registry` | `io.openleap.ops.registry` | Eureka service discovery |
| `io.openleap.registry-repo` | `io.openleap.ops.registry-repo` | Registry configuration |
| `io.openleap.gateway` | `io.openleap.ops.gateway` | API Gateway |
| `io.openleap.debezium` | `io.openleap.ops.debezium` | Change data capture tooling |

### `ui` — Shared UI

| Current | New | Notes |
|---------|-----|-------|
| `io.openleap.ui.base` | — | Already correct |
| `io.openleap.ui.common` | — | Already correct |
| `io.openleap.ui.kit` | — | Already correct |
| `io.openleap.ui.starter` | — | Already correct |
| `io.openleap.ui.guidelines` | — | Already correct |
| `io.openleap.admin.ui` | `io.openleap.ui.admin` | Flip to match `ui.*` pattern |
| `io.openleap.elara.ui` | `io.openleap.ui.elara` | Flip to match `ui.*` pattern |
| `io.openleap.dashboard` | `io.openleap.ui.dashboard` | If active |

### `ai` — AI & Automation

| Current | New | Notes |
|---------|-----|-------|
| `io.openleap.ai.autoclaude` | — | Already correct |
| `io.openleap.ai.autoclaude.ui` | — | Already correct |

---

## Repos to Archive

~30 stale, legacy, or superseded repos should be archived:

**No `io.openleap` prefix (POCs, tools, forks):**
- `crypto-vault-service`, `keycloak-theme`, `nightagenttest`, `rabbitmq-keycloak-auth-poc`
- `message-processor-service`, `message-receiver-service`, `message-receiver-service-client`, `message-receiver-service-legacy-client`, `message-receiver-service-seeder`
- `docker-container-availability-exporter`, `docker-events-exporter`, `filebeat`
- `spring-azure-mail`, `testme-service`, `ubuntu-ssh-temurin`, `dtp-flutter`
- `iam-poc`, `opa-sdk`, `devops`
- `iso-639`, `iso3166`

**Superseded by newer suite repos:**
- `io.openleap.accounting` → replaced by `fi.acc`
- `io.openleap.accounting.account` → replaced by `fi.coa`
- `io.openleap.accounting.ledger` → replaced by `fi.gl`
- `io.openleap.authorization` → replaced by `iam.authz`
- `io.openleap.identity` → replaced by `iam.principal`
- `io.openleap.iam` → replaced by `iam.*` suite
- `io.openleap.report` → replaced by `t1_rpt`
- `io.openleap.ui` → replaced by `ui.*` suite
- `io.openleap.admin` → replaced by `admin.ui`

---

## Spec Repo Split

The current `io.openleap.spec` repo holds **1,601 files** serving 6 distinct purposes with different audiences and change cadences. It is split into 4 focused repos:

### `io.openleap.dev.hub` — Central Documentation Hub (NEW)

The entry point for anyone joining or navigating OpenLeap. Answers "what is this?" and "where do I find X?"

**Contains:**
- What is OpenLeap (vision, architecture overview)
- Repository naming strategy (this document)
- Repo landscape & catalog (from `landscape/REPO_LANDSCAPE.md`, `REPO_CATALOG.yaml`)
- Integration map (from `landscape/INTEGRATION_MAP.md`)
- Platform overview (from `landscape/GESAMTSICHT.md`)
- System overview diagrams (from `spec/SYSTEM_OVERVIEW.md`)
- Status dashboards (from `landscape/specification-status.json`, `implementation-status.json`)
- Links to all other repos by category

### `io.openleap.dev.spec` — Domain & Service Specifications

The formal, authoritative specifications for all platform services and business domains. Pure spec content, no tooling.

**Contains (from `spec/`):**
- `T1_Platform/` — IAM, PARAM, TECH service specs (255 files)
- `T2_Common/` — two T2 suites: `shared` (bp, cap — master data) and `auto` (ntf, wf — automation fabric)
- `T3_Domains/` — CRM, FI, SD, PS, SRV, CO, PPS, OPS, COM, FAC, HR, TKS
- `T4_Data/` — BI/Data specs

### `io.openleap.dev.concepts` — Specification Framework (NEW)

The meta-layer that defines HOW to write specifications. Stable and normative — changes rarely and deliberately.

**Contains (from `concepts/`):**
- `CONCEPTUAL_STACK.md` — SPLE model, platform/product spaces
- `ARTIFACT_CATALOG.md` — Normative definitions of all artifact types
- `RECONCILIATION.md` — Mapping of 4-Tier, 5-Layer, 3-Space models
- `templates/` — Authoring templates (suite-spec, domain-service-spec, feature-spec, etc.)
- `template-registry.json` — Machine-readable template metadata
- `artifacts/` — 31 artifact type definitions (platform + product space)
- `examples/` — Canonical examples (~80 files)
- `governance/` — Template governance, BFF guideline, suite packaging blueprint

### `io.openleap.dev.prompts` — AI Automation & Tooling (NEW)

AI prompts for batch operations on specs and repos, plus the scripts and crawlers that support them.

**Contains (from `prompts/` + `scripts/`):**
- AI prompts: CEO orchestrator, implement, QA, devops, upgrade, crawl (9 files)
- Scripts: crawlers, upgrade scripts, hooks, Docker tooling (36 files)

### Migration Summary

| Source (in current spec repo) | Target Repo | Files |
|-------------------------------|-------------|-------|
| `landscape/*` | `dev.hub` | 11 |
| `spec/OPENLEAP_PLATFORM_GENERAL.md` | `dev.hub` | 1 |
| `spec/SYSTEM_OVERVIEW.md` | `dev.hub` | 1 |
| `spec/T1_*` through `spec/T4_*` | `dev.spec` | ~1,413 |
| `concepts/*` | `dev.concepts` | 124 |
| `prompts/*` | `dev.prompts` | 9 |
| `scripts/*` | `dev.prompts` | 36 |
| `docs/*` | `dev.hub` | 1 |
| New content (README, naming strategy, etc.) | `dev.hub` | ~5 |

### Cross-References

- `dev.hub` links to everything — it's the index
- `dev.spec` READMEs reference `dev.concepts` for template definitions
- `dev.prompts` references `dev.spec` (the target it operates on) and `dev.hub` (for landscape status output)
- `dev.concepts` is standalone and referenced by `dev.spec`

---

## Complete `dev.*` Namespace

After all renames and the spec repo split:

```
io.openleap.dev.hub          — Central documentation & navigation
io.openleap.dev.spec         — Domain & service specifications
io.openleap.dev.concepts     — Specification framework & templates
io.openleap.dev.prompts      — AI automation prompts & scripts
io.openleap.dev.guidelines   — Development guidelines
io.openleap.dev.mvp          — MVP specifications
io.openleap.dev.parent       — Parent POM
io.openleap.dev.starter      — Spring Boot starter
io.openleap.dev.template     — Microservice scaffold
io.openleap.dev.archetype    — Maven archetype
io.openleap.dev.bruno        — API test collections
io.openleap.dev.customize    — Customization tooling
```

---

## `prod.*` Namespace

Product repos follow a three-repo pattern per product. See [architecture/product-repo-layout.md](../architecture/product-repo-layout.md) for the full layout.

```
io.openleap.prod.{product}           — Product specification (Git-reviewed)
io.openleap.prod.{product}.ui        — Product frontend (Nuxt 4 / Vue 3)
io.openleap.prod.{product}.bff       — Product BFF (optional)
```

`{product}` is a short kebab-case identifier (e.g. `acme-sales`). In-house platform tooling UIs (`elara.ui`, `telos.ui`, `iam.ui`, `tech.dms.ui`, etc.) predate this convention and remain under their existing namespaces — `prod.*` applies to customer- or market-derived products going forward.

---

## Design Rationale

### Why `dev` + `ops` and not a single infix like `meta` or `org`?

A single catch-all infix would group ~25 repos under one flat namespace. The split into `dev` (what you need to build) vs `ops` (what you need to run) mirrors how teams naturally think and keeps the namespace scannable.

### Why not more infixes (e.g. `doc`, `tpl`, `infra`, `cicd`)?

Diminishing returns. With 5+ infixes every new repo triggers a debate about which infix it belongs under. Two new infixes cover the two natural clusters without overhead.

### Why keep `ui.guidelines` under `ui` instead of `dev`?

It's a standards document, but it's specifically about UI. A developer looking for UI guidance would look in the `ui.*` namespace, not `dev.*`.

---

## Open Items

1. **`t1.*` vs `tech.*` unification** — Both are used for platform-level services. Worth consolidating under a single prefix in a follow-up.
2. **`io.openleap.core` and `io.openleap.jump-lib`** — Verify whether these are still in active use. Archive if superseded, rename to `dev.*` if active.
3. **`io.openleap.event.cdc` vs `io.openleap.debezium`** — `debezium` is infra tooling (→ `ops`). `event.cdc` is domain-adjacent and stays under `event.*`.
