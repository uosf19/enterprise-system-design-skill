---
name: enterprise-system-design
description: Use when designing, reviewing, or implementing offline-first POS or ERP workflows, tenant boundaries, financial or inventory ledgers, cross-client contracts, or shared design systems with RTL and touch requirements. Skip for isolated styling or standalone CRUD without those concerns.
---

# Enterprise System Design & Multi-Stack Architecture Skill

This skill covers offline-first POS, enterprise data boundaries, and cross-platform design systems. It combines study notes, documented invariants, and verification scenarios. Check [source evidence](references/source-evidence.md) before attributing a specific claim to a book.

---

## Scope & Assumptions

- **Primary domain:** multi-tenant, offline-first restaurant/retail SaaS (POS, KDS, manager mobile, admin web).
- **Stack examples:** Next.js, Tauri, Flutter, and PostgreSQL are illustrative; the patterns are stack-agnostic.
- **Project precedence:** Use the project's current architecture, operating decisions, and release gates for implementation. In Universal ERP, start with `docs/project/erp-master-plan.md` and its topic map. Examples in this skill apply only where compatible with those decisions. Record material gaps or conflicts before changing behavior.
- **Regulatory examples:** ETA, Paymob, Fawry, and similar references are illustrative, not legal or tax advice. Verify current rules for each jurisdiction and document type.
- **Not a complexity mandate:** simple CRUD or low-scale applications should not adopt DDD aggregates, outbox, or event sourcing without a real consistency, audit, or scale requirement.
- **Language note:** Arabic glosses in the references support MENA teams; they are explanatory, not required terminology.
- **Companion skill:** for code-level quality (naming, refactoring, tests) use `code-craftsmanship`. Where the two meet: "crash early" there means abort the faulty operation and quarantine it here (a POS must keep serving other sales); and input validation at the boundary there complements, not replaces, the aggregate invariants here.

### Known Gaps (not covered in depth)
This skill focuses on POS, offline sync and design systems. The [architecture checklist](checklists/architecture-readiness.md) lists additional ERP concerns, including period close, fiscal numbering, costing, access scope, reporting and effective-dated rules. Apply them when the corresponding workflow is in scope and use the approved accounting or provider policy.

---

## Source Notes & Attribution

This skill separates three kinds of material:

1. **Book frameworks and named patterns** — attributed to the source book (e.g., Meadows' leverage points, Frost's atomic levels, Fowler's Money/Unit of Work/Optimistic Offline Lock, Alexander's architectural patterns, EIP's messaging patterns).
2. **External standards and platform guidance** — ISO 4217, WCAG, Semantic Versioning, Apple HIG, and Material Design.
3. **Derived guidance for this skill** — restaurant/POS workflow patterns, component lifecycle stages, token tiers, spacing scales and physical-device test heuristics. Validate implementation choices against the project's constraints.

Where a section mixes these, the text labels the derived material.

---

## How to Apply This Skill

1. **Classify the task.**
   - Domain modeling, ledgers and money → Pillar 1, [architecture review](checklists/architecture-readiness.md), and [Money and persistence](references/13-patterns-of-enterprise-application-architecture.md).
   - Offline sync and integration → Pillar 2, [offline workflow](references/offline-pos-workflow.md), and [sync checklist](checklists/offline-sync-resilience.md).
   - Shared UI, RTL and touch → Pillars 3–4 and the [design checklist](checklists/design-system-tokens.md). Open the references for the specific patterns under review.
   - Architecture review → [architecture checklist](checklists/architecture-readiness.md), with the sync or design checklist where the scope includes those concerns.
2. **Read progressively.** Use this file first; open only the relevant reference(s) and checklist for the task.
3. **Make decisions explicit.** State the chosen pattern, its trade-offs, and any unresolved risks. Do not silently apply a pattern that the context does not justify.
4. **Verify before approval.** Run the relevant checklist before approving a design, RFC, or pull request.

---

## The Four Architectural Pillars

```text
       [ Enterprise Reference Architecture ]
                          |
   +--------------+---------------+--------------+----------------+
   |              |               |              |                |
[ PILLAR 1 ]   [ PILLAR 2 ]    [ PILLAR 3 ]   [ PILLAR 4 ]
Domain &       Distributed     Atomic Cross-  Ergonomics &
Systemic       Data & Sync     Platform       Spatial Pattern
Invariants     Engine          Design System  Language
(Systems + DDD) (DDIA + EIP)    (Tokens + UI)  (Touch + Context)
```

---

## Pillar 1: Domain & Systemic Invariants
*Foundations: systems thinking, Implementing DDD (Vernon), and Patterns of Enterprise Application Architecture (Fowler).*

*Source note: Brown and Terrapin's primer is the local evidence for stocks, flows, and delays. Vernon supplies bounded contexts and aggregates. Fowler supplies Money, Unit of Work, and Optimistic Offline Lock. The ledger design below is domain guidance.*

### 1. The Stock & Flow Ledger Invariant
- **Rule:** The ledger of movements is the source of truth; a balance or quantity is never the only record of a change. A stored balance is allowed only as a **projection** that is updated in the same transaction as the ledger insert (or recomputed from it) and can be reconciled against it. Never overwrite a balance without a corresponding ledger entry.
- **Why a projection is often needed:** summing a ledger cannot by itself prevent overdrafts under concurrency; a locked balance row or serializable transaction does.
- **Implementation:** Balances are derived projections of an immutable, append-only ledger of historical movements:
  ```text
  Stock Balance(t) = Σ Inflows − Σ Outflows
  ```
- **Double-Entry Balance:** When the scope includes a double-entry journal, enforce debit-credit equality inside the posting transaction under its currency rules. Operational cash and stock ledgers follow their own invariants and can ship before general-ledger posting.

### 2. The Money Pattern
*Fowler's Money pattern is the amount-plus-currency abstraction; the integer-minor-unit storage rule below is this skill's implementation standard.*
- **Posted amounts:** Store integer minor units with currency. Use the currency scale when rendering. Validate currency and the supported signed amount range at boundaries and before persistence.
- **Arithmetic and contracts:** Keep rates and intermediate products exact (for example, `bigint` and integer ratios, or a decimal type with explicit precision and rounding). JSON can carry canonical integer strings with validated range across languages. Use JSON numbers only with an explicit safe range for values and any arithmetic done as `number`.
  ```typescript
  interface Money {
    readonly amountMinor: bigint; // 15050n represents 150.50 EGP
    readonly currency: string;   // validated ISO 4217 code
  }
  ```
- **Rounding and allocation:** Specify rounding stages, signed tie behavior, and stable remainder ordering from the applicable policy. Preserve the amount and rule revisions applied to each sale. Share exact vectors for limits, signed values, ties, and conservation of totals (for example, 100.00 split into 33.34 + 33.33 + 33.33). Use synthetic fiscal values until the relevant rules are verified.

### 3. Bounded Context & Aggregate Boundaries
- **Rule:** Identify the business facts that must commit or roll back together. Model small aggregate boundaries around immediate invariants and document any transaction spanning them. Coordinate only effects that tolerate delay through domain events and the outbox. For the Universal ERP cash sale, save the sale, tender, immutable receipt data, print job and sync command in one local transaction. Dispatch printing after commit.
- **Reference by identity:** Aggregates hold other aggregates' IDs, not object references (Vernon). Order lines store the item ID plus a snapshot of the price and tax rule applied at sale time.
- **Anti-Corruption Layer (ACL):** External vendor payloads (payment gateways, tax authorities, biometric clocks) must be parsed and mapped through an Anti-Corruption Layer before entering the domain model.

---

## Pillar 2: Distributed Data & Resilient Sync Engine
*Foundations: Designing Data-Intensive Applications (Kleppmann), Enterprise Integration Patterns (Hohpe).*

### 1. The Transactional Outbox Pattern
*The named Transactional Outbox pattern is commonly credited to Chris Richardson's microservices catalog. DDIA analyzes the underlying dual-write and CDC problems but does not present this as a named DDIA pattern. The SQL below is an implementation for this skill.*
- **Rule:** Domain state mutations and their outbound event records must be committed atomically inside the exact same ACID database transaction:
  ```sql
  -- Atomic domain write + outbox event
  -- PostgreSQL 13+; on older versions enable pgcrypto for gen_random_uuid()
  BEGIN;
  INSERT INTO sales_orders (...) VALUES (...);
  INSERT INTO outbox_events (id, event_type, payload, status)
  VALUES (gen_random_uuid(), 'sales.order_placed.v1', :payload, 'pending');
  COMMIT;
  ```
- An outbox worker can process records from the same database. Add a broker only for a justified delivery need. Give effects stable IDs and make consumers idempotent across worker retries.

### 2. Deterministic Idempotency Key Invariant
- **Rule:** Enforce a unique, contract-defined retry identity. For Universal ERP POS operations:
  ```text
  Idempotency Key = (organization_id, device_id, device_epoch, operation_id)
  ```
  For other integrations, scope the key to the authenticated tenant and source under their contract.
- Commit the key, versioned request fingerprint, recorded outcome, domain effect and any outbox record together. After checking the caller's scope, replay the original outcome for a matching request. Route the same key with different content to review without reapplying effects. Define which contract fields are hashed and how they are canonicalized.
- Set retention from supported outages, backup recovery and late arrivals. Keep stable business references (sale or receipt ID, transfer receipt cycle) to reconcile across different keys, expired retry records or a new epoch.

### 3. Monotonic Logical Sequencing & Conflict Resolution
- **Rule:** Never rely on device wall-clock timestamps for distributed transaction ordering.
- **Implementation:**
  - Clients emit `local_sequence` scoped to device and epoch. When restoring or reinstalling, reconcile the prior history and pending identities before issuing new work in a new epoch. Give each enrolled device its own identity.
  - Per-device counters order one device's operations only; they do not give a causal order across devices.
  - Recover gaps in the device sequence. Record business rejections as outcomes and continue independent work under the project policy. Apply pulled updates and their cursor atomically before acknowledgement.
  - Keep distinct claims as evidence. Post their effects only after domain deduplication and conflict handling. Two equivalent confirmations of the same transfer receipt cycle count once even with different retry keys. Conflicting counts or shipment revisions enter review under that cycle.
  - Editable entity conflicts affecting money, stock, or compliance are isolated into an explicit operator review queue (`AttendanceCase`, `ReconciliationException`), never resolved via silent Last-Write-Wins (LWW). Reserve LWW for non-critical fields with a documented classification.

### 4. Dead-Letter Queue (DLQ) & Quarantine Isolation
- Retry transient failures with bounded exponential backoff and jitter. Route invalid data and business rejections to a recorded outcome or authorized review immediately. Verify an uncertain payment or print against its stable reference before repeating an external action. For an already paid or issued sale, preserve the original and correct through an authorized linked refund or adjustment. Keep unrelated work moving.

---

## Pillar 3: Atomic Cross-Platform Design Systems
*Foundations: Atomic Design (Frost), Frontend Architecture (Godbolt), Design Systems Handbook (InVision), Laying the Foundations (Couldwell).*

### 1. The Platform-Agnostic Token Architecture
*The three-tier hierarchy below is a common implementation model, not a universal book mandate.*
- Author all design decisions in a single JSON/YAML token registry (`packages/design-tokens`):
  - **Global Tokens:** Primitive palette (`color.palette.emerald.500: #10b981`).
  - **Semantic Tokens:** Purpose-bound (`color.status.positive: {color.palette.emerald.500}`).
  - **Component Tokens:** Component-scoped (`badge.background.success: {color.status.positive}`).
- Automated CI pipeline (Style Dictionary or equivalent) compiles tokens into:
  - Tailwind configuration or CSS Custom Properties for Web & Desktop (Next.js & Tauri).
  - Flutter theme extensions or constants for Mobile (Flutter).

### 2. The 4px/8px Spatial Rhythm & Bidirectional (RTL/LTR) Rules
- **Grid Rule (skill standard):** Layout spacing and component dimensions should use the 4px/8px scale. Hairline borders (1px), focus rings (2px), and optical corrections are documented exceptions; linters should allow them explicitly.
- **Directional Agnosticism:** Use **CSS Logical Properties** for directional layout where supported:
  - Use `margin-inline-start` instead of `margin-left`.
  - Use `padding-inline-end` instead of `padding-right`.
  - Use `inset-inline-start` instead of `left`.
  - Properties without logical equivalents (e.g., some transforms and background positions) need direction-aware helpers.

### 3. Dual-Tier System Governance
*This two-level split is a pragmatic simplification; Kholmatova treats strictness as a spectrum.*
- **Strict Governance:** Mission-critical transactional interfaces (POS checkout, billing, payments, tax compliance) enforce strict component usage; no inline CSS or untokenized overrides permitted.
- **Loose Governance:** Reporting, analytics dashboards, and internal exploration allow composable layout variation while honoring base tokens.

---

## Pillar 4: Ergonomics & Spatial Pattern Language
*Foundations: A Pattern Language (Alexander), Designing for Touch (Clark), Expressive Design Systems (Perez-Cruz).*

### 1. Touch Target Geometry & Spacing
- **Platform Baselines:** Apple's HIG recommends a 44×44 pt hit region; Material Design recommends 48×48 dp. For POS/KDS, use at least 48×48 CSS px where practical, and validate the physical size on target hardware; CSS px is not a fixed physical unit.
- **Safety Buffer:** Adjacent touchable targets should maintain at least 8px separation, adapting Material's 8dp guidance, to reduce accidental mis-taps during peak operating hours; validate on the target device.

### 2. Environmental Context Adaptation

| Surface | Operating Environment | Architectural Requirements |
| :--- | :--- | :--- |
| **POS Terminal** | Standing operator, rush workload, glare. | Keep totals and primary actions in reach. Measure feedback, tap accuracy, and keyboard access on the target layout in both directions. Separate tender and cancellation. |
| **Kitchen Display (KDS)** | Viewing distance, steam, glare, and limited reach vary. | Validate legibility and controls at the actual distance. Label status in text as well as color and test keyboard or bump-bar operation where present. |
| **Manager Mobile** | On-the-go, one- or two-handed grip. | Test reach in both hands. Separate destructive actions from frequent actions and use the confirmation or authorization required by the workflow. |
| **Admin Web Console** | Desktop monitor, office environment, mouse & keyboard. | High data density, sortable tables, keyboard navigation shortcuts, multi-column filters. |

### 3. Spatial Restaurant & Retail Patterns
Alexander's architectural patterns that inform this section include **Street Cafe (88)**, **Food Stands (93)**, **Intimacy Gradient (127)**, **Eating Atmosphere (182)**, and **Cooking Layout (184)**. The following software patterns are derived for this skill, not Alexander pattern names:
- **Linear Checkout Flow (derived):** Fast, linear checkout (`Items -> Modifiers -> Subtotal -> Cash -> Receipt -> Drawer Kick`).
- **Expo Synchronization View (derived):** Dedicated expo screen synchronizing multi-station preparation (Hot Kitchen + Bar + Salad) before marking an order ready for pickup.
- **Blind Shift Count (optional):** Showing the expected amount only after counting is a policy option. Follow the approved procedure for the drawer and keep expected, counted, and variance amounts auditable.

---

## Architectural Decision Checklist

Use the [architecture checklist](checklists/architecture-readiness.md) for applicable domain and system concerns. Use the [sync checklist](checklists/offline-sync-resilience.md) for client durability and the [design checklist](checklists/design-system-tokens.md) for shared interfaces.

---

## References & Checklists

### Book References
- [01. Thinking in Systems (Donella Meadows)](references/01-thinking-in-systems.md)
- [02. Atomic Design (Brad Frost)](references/02-atomic-design.md)
- [03. Design Systems (Alla Kholmatova)](references/03-design-systems-kholmatova.md)
- [04. Design Systems Handbook (InVision)](references/04-design-systems-handbook.md)
- [05. Expressive Design Systems (Yesenia Perez-Cruz)](references/05-expressive-design-systems.md)
- [06. Laying the Foundations (Andrew Couldwell)](references/06-laying-the-foundations.md)
- [07. Building Design Systems (Sarrah Vesselov)](references/07-building-design-systems.md)
- [08. Practical UI Patterns (Diana MacDonald)](references/08-practical-ui-patterns.md)
- [09. Frontend Architecture (Micah Godbolt)](references/09-frontend-architecture.md)
- [10. A Pattern Language (Christopher Alexander)](references/10-a-pattern-language.md)
- [11. Designing Data-Intensive Applications (Martin Kleppmann)](references/11-designing-data-intensive-applications.md)
- [12. Implementing Domain-Driven Design (Vaughn Vernon)](references/12-implementing-domain-driven-design.md)
- [13. Patterns of Enterprise Application Architecture (Martin Fowler)](references/13-patterns-of-enterprise-application-architecture.md)
- [14. Enterprise Integration Patterns (Gregor Hohpe)](references/14-enterprise-integration-patterns.md)
- [15. Designing for Touch (Josh Clark)](references/15-designing-for-touch.md)

### Operational Checklists
- [Architecture Readiness](checklists/architecture-readiness.md)
- [Offline-First & Sync Resilience](checklists/offline-sync-resilience.md)
- [Design System & Tokens](checklists/design-system-tokens.md)
