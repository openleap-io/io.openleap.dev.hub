# Getting Started — From Product Idea to Release

This document walks a product idea all the way to a released configuration. It is the practical counterpart to [sple-platform-concept.md](sple-platform-concept.md) (the model) and [product-repo-layout.md](product-repo-layout.md) (the repo shape).

Two worked examples are used throughout:

- **CRM Workbench** — a catalog-rich case: the CRM suite is mature, so most discovery needs match existing features.
- **Ticket Desk** — a catalog-incomplete case: only `srv.cas` exists in the SRV suite, so discovery surfaces gaps that must be either promoted to platform features or filled at extension points.

> **Feature IDs in this document are illustrative.** Values like `F-CRM-001 "Manage Leads"` are placeholders used for narrative continuity. Reconcile against the real `dev.spec` catalog before authoring a production product spec.

---

## 1. The Path at a Glance

Six phases, mapped to [sple-platform-concept.md §4.5](sple-platform-concept.md). The phases are sequential per feature but the overall product moves through them iteratively as the catalog and the product co-evolve.

```
┌────────────────────┐   ┌────────────────────┐   ┌────────────────────┐
│  1. Discovery      │──▶│  2. Gap Analysis   │──▶│  3. Draft Spec     │
│  Elara             │   │  candidate vs      │   │  prod.{product}    │
│  personas, process │   │  dev.spec catalog  │   │  selection.uvl     │
└────────────────────┘   └─────────┬──────────┘   └─────────┬──────────┘
                                   │                        │
                                   │ gaps?                  │
                                   ▼                        ▼
                         ┌────────────────────┐   ┌────────────────────┐
                         │  Promote new       │   │  4. Configure      │
                         │  feature to        │   │  variability       │
                         │  dev.spec          │   │  bindings.yaml     │
                         └────────────────────┘   └─────────┬──────────┘
                                                            │
                                                            ▼
                                                  ┌────────────────────┐
                                                  │  5. Extend         │
                                                  │  extensions/F-*/   │
                                                  └─────────┬──────────┘
                                                            │
                                                            ▼
                                                  ┌────────────────────┐
                                                  │  6. Release        │
                                                  │  bff/productconfig │
                                                  │  UI + BFF deploy   │
                                                  └────────────────────┘
```

### Phase reference

| Phase | What you do | Primary tool | Output artifact |
|-------|-------------|--------------|-----------------|
| 1. Discovery | Interview stakeholders; identify business actors, personas, processes, goals | Elara | Personas (draft), BPMN, Feature Candidates |
| 2. Gap Analysis | Match each Feature Candidate against `dev.spec` catalog | Elara + `dev.spec` | Matched list; Gap list |
| 3. Draft Spec | Create `io.openleap.prod.{product}` with `product.yaml`, `features/selection.uvl`, personas, processes | Git + Elara export | Product repo on feature branch |
| 4. Configure | Resolve UVL attributes per binding time (compile / deploy / runtime) | Elara UVL editor | `variability/bindings.yaml` |
| 5. Extend | Fill platform-declared extension points with product-specific logic | Editor + Elara | `extensions/F-*/…` |
| 6. Release | Package `bff/productconfig.yaml`; build + deploy UI and BFF | CI | Running system |

Gap handling (between phases 2 and 3) is the decision point covered in §5 below.

---

## 2. Worked Example A — CRM Workbench (Catalog-Rich)

The customer: a mid-market services company. They want a single workspace where their sales team manages leads, contacts, opportunities, activities, and outbound email — with visibility into customer invoice history.

### A.1 Discovery (in Elara)

Interview output, captured in Elara:

**Personas**

| Persona | Goal | Daily tasks |
|---------|------|-------------|
| `account-executive` | Close deals | Triage inbound leads, log activities, advance opportunity stages, send email sequences |
| `sales-manager` | Forecast & coach | Review pipeline, inspect activities, reassign leads |

**Process** — `lead-to-close.bpmn`:

```
Inbound Lead → Qualify → Create Opportunity → Log Activities → Advance Stage → Close (Won | Lost)
```

**Feature Candidates** produced by Elara:

1. Manage Leads
2. Manage Contacts
3. Manage Opportunities
4. Log Activities
5. Send Outbound Email
6. Pipeline Dashboard (manager view)
7. See Customer Invoice History (cross-suite read into FI)
8. See Business Partner master data (cross-suite read into BP)

### A.2 Gap Analysis

| Candidate | Suite | Existing feature? | Action |
|-----------|-------|-------------------|--------|
| Manage Leads | CRM | `F-CRM-001` (illustrative) | Select |
| Manage Contacts | CRM | `F-CRM-002` | Select |
| Manage Opportunities | CRM | `F-CRM-003` | Select |
| Log Activities | CRM | `F-CRM-004` | Select |
| Send Outbound Email | CRM | `F-CRM-005` | Select |
| Pipeline Dashboard | CRM | `F-CRM-010` (assumes an analytics feature exists) | Select |
| Customer Invoice History | FI | `F-FI-007` (cross-suite read, read-only inclusion) | Select read-only |
| Business Partner master data | BP (T2) | `F-BP-001` (T2 shared kernel) | Select |

All candidates map to the existing catalog. No gaps. Proceed to §A.3.

### A.3 Drafting `io.openleap.prod.crm-workbench`

The layout matches [product-repo-layout.md §3](product-repo-layout.md).

**`product.yaml`**

```yaml
id: prod.crm-workbench
name: CRM Workbench
owner: team-sales-platform
target-customer: acme-services
version: 0.1.0
personas:
  - account-executive
  - sales-manager
processes:
  - lead-to-close
```

**`features/selection.uvl`**

```uvl
features
    Product "CRM Workbench"
        mandatory
            F-CRM-001  "Manage Leads"
            F-CRM-002  "Manage Contacts"
            F-CRM-003  "Manage Opportunities"
            F-CRM-004  "Log Activities"
            F-BP-001   "Business Partner"          // T2 — freely included
        optional
            F-CRM-005  "Send Outbound Email"
            F-CRM-010  "Pipeline Dashboard"
            F-FI-007   "Customer Invoice History"  // cross-suite, read-only

    inclusion-modes
        F-FI-007 : read-only                        // mutate-local rule, see §5 below

    constraints
        F-CRM-001 requires F-BP-001                 // leads need a BP record
        F-CRM-003 requires F-CRM-002                // opportunities need contacts
```

**`variability/bindings.yaml`**

```yaml
bindings:
  F-CRM-001:
    lead-scoring-model: "linear"        # attribute; binding: compile
    max-open-leads-per-rep: 200         # binding: deploy
  F-CRM-005:
    email-provider: "smtp"              # binding: deploy
    daily-send-cap: 500                 # binding: runtime
```

**Small extension — custom field on Lead**

`F-CRM-001` declares an extension point `custom-fields` on the Lead aggregate. Acme wants to capture `source-campaign-id`:

`extensions/F-CRM-001/fields.yaml`:

```yaml
aggregate: Lead
custom-fields:
  - name: source-campaign-id
    type: string
    required: false
    index: true
    ui:
      label: "Campaign"
      zone: lead-entry.details
```

**Personas and processes**

`personas/account-executive.yaml`:

```yaml
id: account-executive
display-name: Account Executive
goals:
  - close-deals
  - hit-monthly-quota
top-tasks:
  - triage-inbound-leads
  - log-activities
  - advance-opportunity-stage
skill-level: intermediate
device-mix: { desktop: 0.8, mobile: 0.2 }
```

`processes/lead-to-close.bpmn` — standard BPMN 2.0 file (referenced from the feature flow).

### A.4 Configure and Release

1. **Configure variability** in Elara UVL editor — it validates `bindings.yaml` against the UVL model in `dev.spec`.
2. **CI packages** `bff/productconfig.yaml` from the selection + bindings + extensions:

```yaml
product-id: prod.crm-workbench
version: 0.1.0
features:
  F-CRM-001: { mode: full,      bindings: { lead-scoring-model: linear, max-open-leads-per-rep: 200 } }
  F-CRM-002: { mode: full }
  F-CRM-003: { mode: full }
  F-CRM-004: { mode: full }
  F-CRM-005: { mode: full,      bindings: { email-provider: smtp, daily-send-cap: 500 } }
  F-CRM-010: { mode: full }
  F-BP-001:  { mode: full }
  F-FI-007:  { mode: read-only }
extensions:
  F-CRM-001:
    custom-fields: { source-campaign-id: { type: string, index: true } }
```

3. **Build and deploy** `io.openleap.prod.crm-workbench.ui` (reads from its paired BFF, not platform services directly).
4. **At runtime**, the BFF loads `productconfig.yaml`, enforces feature-gating (per [sple-platform-concept.md §4.2](sple-platform-concept.md)), and routes extension calls.

---

## 3. Worked Example B — Ticket Desk (Catalog-Incomplete)

The customer: a managed-services provider whose agents resolve customer issues against SLAs, using a knowledge base and a self-service customer portal.

### B.1 Discovery (in Elara)

**Personas**

| Persona | Goal | Daily tasks |
|---------|------|-------------|
| `support-agent` | Resolve tickets within SLA | Triage queue, work tickets, consult KB, escalate |
| `support-supervisor` | Meet team SLAs | Monitor queues, reassign, review KB quality |
| `end-customer` | Get help fast | Submit tickets, track status, search KB |

**Process** — `ticket-lifecycle.bpmn`:

```
Submit → Triage → Assign → Work
                    │
                    └─► Escalate (if SLA at risk) → Re-assign → Work
                                                                  │
                                                                  ▼
                                                                Resolve → Customer confirms → Close
```

**Feature Candidates**:

1. Create Case
2. Assign Case
3. Work Case (notes, status transitions)
4. Resolve Case
5. **SLA Management** (measure time-to-first-response, time-to-resolution; trigger escalation)
6. **Knowledge Base** (articles, search, link to case)
7. **Customer Portal** (end-customer self-service)
8. **Escalation Routing** (policy-driven reassignment)

### B.2 Gap Analysis

| Candidate | Suite | Existing feature? | Action |
|-----------|-------|-------------------|--------|
| Create Case | SRV | `F-SRV-001 "Manage Cases"` (on `srv.cas`) | Select |
| Assign Case | SRV | `F-SRV-001` (included) | Covered |
| Work Case | SRV | `F-SRV-001` (included) | Covered |
| Resolve Case | SRV | `F-SRV-001` (included) | Covered |
| **SLA Management** | SRV | **Gap** | Decide: promote vs extend |
| **Knowledge Base** | SRV | **Gap** (likely a new domain, `srv.kb`) | Promote |
| **Customer Portal** | SRV | **Gap** (cross-cutting auth + case-read view) | Promote |
| **Escalation Routing** | SRV | **Gap** | Decide: promote vs extend |

Four gaps. Two obvious promotes (KB, Customer Portal — both are broad enough to be reused by other products in the future). Two need the decision framework (SLA, Escalation Routing).

### B.3 Responding to Gaps

#### B.3.a Promote to platform — Knowledge Base

Knowledge Base is not product-specific. Any product using SRV could want it. It deserves a proper platform feature.

**Workflow:**

1. Draft a new feature spec in `dev.spec` under the SRV suite (new domain `srv.kb` if needed). This includes `feature-spec.md`, `feature-model.uvl`, `screen-contract.aui`, `screen-contract.cui` — the four-artifact bundle defined in [sple-platform-concept.md §4.1](sple-platform-concept.md).
2. Open a PR against `dev.spec` with the SRV suite owners as reviewers.
3. Once merged, reference the new feature `F-SRV-020 "Knowledge Base"` in `prod.ticket-desk/features/selection.uvl`.
4. The backend team implements `io.openleap.srv.kb` in parallel. The product spec can declare the feature and continue authoring; release is gated on the platform service existing.

#### B.3.b Promote to platform — Customer Portal

Same pattern as KB. Customer Portal is a reusable notion (any SRV product exposing tickets to end-customers will want it). Produces `F-SRV-030 "Customer Portal"`.

#### B.3.c Extend — SLA Management on Cases

SLA rules are product-specific in practice (every customer negotiates their own SLAs). `F-SRV-001` already declares extension points for `rules` and `actions`. Fill them:

`extensions/F-SRV-001/fields.yaml`:

```yaml
aggregate: Case
custom-fields:
  - { name: sla-target-first-response-minutes, type: integer }
  - { name: sla-target-resolution-minutes,     type: integer }
  - { name: sla-breached,                      type: boolean, computed: true }
```

`extensions/F-SRV-001/rules.yaml`:

```yaml
rules:
  - id: sla-first-response-warning
    on: case.status-changed
    when: "now() > created-at + (sla-target-first-response-minutes - 5) minutes AND first-response-at == null"
    then: emit-event("case.sla-first-response-warning")
  - id: sla-breach
    on: case.clock-tick
    when: "now() > created-at + sla-target-resolution-minutes minutes AND resolved-at == null"
    then: set-field("sla-breached", true); emit-event("case.sla-breached")
```

`extensions/F-SRV-001/actions.yaml`:

```yaml
actions:
  - id: reassign-on-breach
    triggered-by: case.sla-breached
    handler: product-service://ticket-desk-escalation-svc/reassign
```

#### B.3.d Extend — Escalation Routing (initially)

Escalation policies are also customer-specific. Start as an extension of `F-SRV-001` using actions routed to a small product-specific service. **Later**, if a second product needs similar routing, promote to a platform feature.

This staged approach — *extend now, promote later when reuse appears* — is explicitly sanctioned by the model: extensions are first-class, and platform features are grown from observed reuse, not guessed.

### B.4 Drafting `io.openleap.prod.ticket-desk`

**`features/selection.uvl`**

```uvl
features
    Product "Ticket Desk"
        mandatory
            F-SRV-001  "Manage Cases"
            F-SRV-020  "Knowledge Base"            // promoted to platform
            F-SRV-030  "Customer Portal"           // promoted to platform
            F-BP-001   "Business Partner"          // T2
            F-IAM-002  "End-customer Auth"         // T1 auth
        optional
            F-CRM-002  "Contacts"                  // cross-suite, read-only (agent sees customer profile)

    inclusion-modes
        F-CRM-002 : read-only

    constraints
        F-SRV-020 requires F-SRV-001               // KB articles link to cases
        F-SRV-030 requires F-IAM-002               // portal needs end-customer identity
        F-SRV-001 requires F-BP-001                // cases belong to a BP
```

**`extensions/F-SRV-001/`** — the SLA and escalation files from §B.3.c / §B.3.d.

**`personas/`**, **`processes/`** — as in §B.1.

### B.5 Release

Same as §A.4 — CI packages `bff/productconfig.yaml`, build deploys UI + BFF. The BFF routes SLA extension events into the product-specific `ticket-desk-escalation-svc`.

---

## 4. Decision Framework — Promote vs Extend

When discovery surfaces a gap, the choice is:

| Criterion | Lean **Promote** (to platform feature) | Lean **Extend** (in product) |
|-----------|----------------------------------------|------------------------------|
| Reusability across products | High — ≥2 products would want it | Low — unique to this customer |
| Suite ownership | Fits a suite's UBL cleanly | Pure customer workflow on top of an existing feature |
| Mutation scope | Writes in the owning suite's domain | Only customer fields, rules, actions |
| Cross-suite reach | Shared kernel material (T2) or new tier-appropriate domain | Stays within one feature's extension points |
| Stability | Requirements stable, broadly understood | Negotiated per customer, may churn |
| Time pressure | Platform team has capacity | Product needs to ship now |

Rule of thumb: **promote when reuse is visible; extend when uniqueness is the point.** A gap extended today can be promoted tomorrow — this is not a one-way door. The reverse (promote then retract) is painful; err toward extension if unsure.

See [sple-platform-concept.md §6](sple-platform-concept.md) for the full extension-point taxonomy (feature / domain / service levels).

---

## 5. Pre-Release Checklist

Before tagging a product release:

- [ ] Discovery complete — personas and processes captured in Elara
- [ ] Gap analysis done — every Feature Candidate either matched, promoted, or extended
- [ ] `prod.{product}` repo created and populated per [product-repo-layout.md §3](product-repo-layout.md)
- [ ] `features/selection.uvl` validates against `dev.spec` (UVL validator resolves all F-IDs and constraints are satisfiable)
- [ ] `variability/bindings.yaml` resolves every required attribute; binding times assigned
- [ ] Extensions only touch declared extension points (no ad-hoc modifications of platform features)
- [ ] Cross-suite references obey **read-across, mutate-local** — other suites' T3 features included only `read-only`; mutations to other suites routed via events
- [ ] `personas/` and `processes/` linked from `product.yaml`
- [ ] `bff/productconfig.yaml` packaged by CI and reviewed
- [ ] Paired `prod.{product}.ui` (and `.bff` if separate) builds green against the packaged config
- [ ] `CHANGELOG.md` updated

---

## 6. Common Pitfalls

1. **Adding net-new logic directly in a product.** Forbidden. Product code must go through declared extension points ([sple-platform-concept.md §6](sple-platform-concept.md)). If no suitable extension point exists, promote a new one in the platform feature — don't fork.
2. **Cross-suite mutations.** Forbidden. A SD feature may *read* from PPS but must not *write* to it. Triggers use domain events; suite B reacts ([sple-platform-concept.md §5](sple-platform-concept.md)).
3. **Forking a platform feature to customize it.** Forbidden. The open-closed principle is strict here. Fork never, extend always.
4. **Bypassing the BFF.** Product UIs call their BFF, never the platform services directly. The BFF is where feature-gating and extension routing happen.
5. **Letting Elara DB diverge from the spec repo.** Elara is the authoring UI, the repo is the system of record. If you edit in Elara and forget to export, the next reviewer sees a stale repo. Export on every material change.
6. **Promoting prematurely.** A gap filled for one customer isn't yet a platform feature. Wait for the second customer (or a credible third) to ask for the same thing. Premature promotion bloats the catalog.
7. **Retracting after promotion.** If a feature made it into `dev.spec` and one product uses it, removing it breaks that product. The platform contract is effectively append-only; deprecate carefully.
8. **Copying platform spec content into the product repo.** The product repo must reference, not copy. Copies rot.

---

## 7. Related Documents

- [sple-platform-concept.md](sple-platform-concept.md) — the two-lifecycle model, specification artifacts, extension-point taxonomy, cross-suite rules.
- [product-repo-layout.md](product-repo-layout.md) — the `io.openleap.prod.{product}` repo shape.
- [integrated-overview.md](integrated-overview.md) — full architecture overview; §8 covers PLE.
- [four-tier-architecture.md](four-tier-architecture.md) — tier and suite boundaries.
- [naming/repo-naming-strategy.md](../naming/repo-naming-strategy.md) — `prod.*` infix and wider naming conventions.
