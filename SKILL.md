---
name: enterprise-system-design
description: Enterprise system design and multi-stack architecture for complex SaaS platforms, offline-first restaurant/retail POS systems, financial and inventory ledgers, domain-driven design, and cross-platform design systems. Informed by 15 books; POS-centered, with ERP back-office gaps listed in the Scope section. Use when designing, reviewing, or implementing enterprise SaaS, POS/retail, offline sync, RTL interfaces, design tokens, or multi-client frontend architecture.
---

# Enterprise System Design & Multi-Stack Architecture Skill

This skill is a production-grade architectural playbook for designing, auditing, and scaling mission-critical SaaS platforms. Its primary focus is offline-first restaurant/retail POS systems and cross-platform design systems, but its invariants apply to other multi-tenant enterprise domains. It is informed by 15 engineering and design books (short study notes in `references/`) and adds derived invariants, heuristics, and checklists, labeled as such.

---

## Scope & Assumptions

- **Primary domain:** multi-tenant, offline-first restaurant/retail SaaS (POS, KDS, manager mobile, admin web).
- **Stack examples:** Next.js, Tauri, Flutter, and PostgreSQL are illustrative; the patterns are stack-agnostic.
- **Regulatory examples:** ETA, Paymob, Fawry, and similar references are illustrative, not legal or tax advice. Verify current rules for each jurisdiction and document type.
- **Not a complexity mandate:** simple CRUD or low-scale applications should not adopt DDD aggregates, outbox, or event sourcing without a real consistency, audit, or scale requirement.
- **Language note:** Arabic glosses in the references support MENA teams; they are explanatory, not required terminology.
- **Companion skill:** for code-level quality (naming, refactoring, tests) use `code-craftsmanship`. Where the two meet: "crash early" there means abort the faulty operation and quarantine it here (a POS must keep serving other sales); and input validation at the boundary there complements, not replaces, the aggregate invariants here.

### Known Gaps (not covered in depth)
This skill is strongest on POS, offline sync, and design systems. For a full ERP, add guidance for: fiscal period close and locking; legally required, gapless document numbering (verify per jurisdiction); inventory costing (FIFO/weighted average); tenancy isolation model; authorization model (RBAC/ABAC); reporting/read-model separation from transactional tables; effective-dated tax and price rules. `checklists/architecture-readiness.md` section 8 lists these as review prompts.

---

## Source Notes & Attribution

This skill separates three kinds of material:

1. **Book frameworks and named patterns** — attributed to the source book (e.g., Meadows' leverage points, Frost's atomic levels, Fowler's Money/Unit of Work/Optimistic Offline Lock, Alexander's architectural patterns, EIP's messaging patterns).
2. **External standards and platform guidance** — ISO 4217, WCAG, Semantic Versioning, Apple HIG, and Material Design.
3. **Derived guidance for this skill** — restaurant/POS workflow patterns, the five-stage component lifecycle, the three-tier token model, exact 4px/8px rules, the 8px web adaptation of Material's 8dp spacing guidance, and 50/100 ms feedback budgets. These are implementation recommendations, not claims about what the source books say.

Where a section mixes these, the text labels the derived material.

---

## How to Apply This Skill

1. **Classify the task.**
   - Domain modeling, ledgers, money → Pillar 1 + references 01, 12, 13 + `checklists/architecture-readiness.md`.
   - Offline sync, idempotency, messaging → Pillar 2 + references 01, 11, 14 + `checklists/offline-sync-resilience.md`.
   - Design tokens, components, RTL → Pillar 3 + references 02–09 + `checklists/design-system-tokens.md`.
   - Touch ergonomics, physical context → Pillar 4 + references 10, 15 + `checklists/design-system-tokens.md`.
   - Architecture review → this file + all three checklists.
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
*Foundations: Thinking in Systems (Meadows), Implementing DDD (Vernon), Patterns of Enterprise Application Architecture (Fowler).*

*Source note: Meadows supplies stocks/flows and leverage points; Vernon supplies bounded contexts and aggregates; Fowler supplies Money, Unit of Work, and Optimistic Offline Lock. Double-entry accounting and the append-only ledger are domain standards adopted for this skill.*

### 1. The Stock & Flow Ledger Invariant
- **Rule:** The ledger of movements is the source of truth; a balance or quantity is never the only record of a change. A stored balance is allowed only as a **projection** that is updated in the same transaction as the ledger insert (or recomputed from it) and can be reconciled against it. Never overwrite a balance without a corresponding ledger entry.
- **Why a projection is often needed:** summing a ledger cannot by itself prevent overdrafts under concurrency; a locked balance row or serializable transaction does.
- **Implementation:** Balances are derived projections of an immutable, append-only ledger of historical movements:
  ```text
  Stock Balance(t) = Σ Inflows − Σ Outflows
  ```
- **Double-Entry Balance:** Every financial journal entry must enforce debit-credit equality (`Σ Debits = Σ Credits`, equivalently `Σ Debits − Σ Credits = 0`) inside the database transaction before commitment.

### 2. The Money Pattern
*Fowler's Money pattern is the amount-plus-currency abstraction; the integer-minor-unit storage rule below is this skill's implementation standard.*
- **Rule:** Floating-point arithmetic is strictly prohibited for monetary representation.
- **Implementation:** Store and calculate all money in integer minor units (e.g., piasters/cents) paired with an explicit ISO 4217 currency code (the minor-unit scale is looked up from the currency, not stored on each value):
  ```typescript
  interface Money {
    readonly amount: number;   // integer minor units (15050 = 150.50 EGP)
    readonly currency: string; // ISO 4217 code: "EGP", "USD", "JPY", "KWD"
    // scale (decimal digits: 2 for EGP/USD, 0 for JPY, 3 for KWD) comes from a currency table keyed by `currency`
  }
  ```
- `number` is safe for typical amounts; for very large ledgers or high-precision aggregation, use `bigint` or a decimal library.
- **Rounding:** Use a documented deterministic policy. Banker's rounding (half-even) is a safe default for aggregated financial calculations; some jurisdictions require half-up. Encode and test the policy once in the domain core. When splitting an amount (split bills, discounts, tax distribution), use an **allocation** routine that distributes the remainder (100.00 / 3 = 33.34 + 33.33 + 33.33) so parts always sum to the whole; Fowler's Money pattern covers this.

### 3. Bounded Context & Aggregate Boundaries
- **Rule:** An Aggregate is the consistency boundary. Prefer one Aggregate instance per database transaction; when multiple aggregates must change together, coordinate them asynchronously via Domain Events and the Transactional Outbox, or explicitly justify the larger transaction.
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
- A background outbox worker polls or streams events from the outbox table to the message broker.

### 2. Deterministic Idempotency Key Invariant
- **Rule:** All retryable write APIs, sync endpoints, and external webhook receivers must enforce idempotency using a unique key, typically scoped to the tenant and client:
  ```text
  Idempotency Key = (organization_id, device_id, operation_id)
  ```
  For server-originated operations, use a stable client or worker ID in place of `device_id`.
- Retrying an identical operation must return the cached original result without re-executing business logic or creating duplicate records.
- Store a fingerprint (hash) of the request payload with the key. The same key with a **different** payload is a client bug: reject it (409/422) instead of returning the old result.
- Define a retention window for stored keys and responses, longer than the maximum offline period you support.

### 3. Monotonic Logical Sequencing & Conflict Resolution
- **Rule:** Never rely on device wall-clock timestamps for distributed transaction ordering.
- **Implementation:**
  - Clients emit a strictly monotonic sequence counter per device (`local_sequence`), scoped by a `device_epoch` that changes whenever the local database is restored, reinstalled, or cloned, so a reset counter cannot collide with old idempotency keys.
  - Per-device counters order one device's operations only; they do not give a causal order across devices.
  - Servers validate the sequence and track cloud changes with opaque sequential cursors.
  - Append-only ledgers converge via idempotent replay and set union.
  - Editable entity conflicts affecting money, stock, or compliance are isolated into an explicit operator review queue (`AttendanceCase`, `ReconciliationException`), never resolved via silent Last-Write-Wins (LWW). Reserve LWW for non-critical fields with a documented classification.

### 4. Dead-Letter Queue (DLQ) & Quarantine Isolation
- External adapter failures (e.g., rejected tax submissions, malformed webhooks) must be isolated into a Dead Letter Queue after capped exponential backoff with jitter (preferably full jitter: `delay_n = random(0, min(cap, base × 2^n))`, e.g., a 5-minute cap; small additive jitter spreads a reconnect storm poorly). Never drop messages or halt the upstream processing queue.

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
| **POS Terminal** | Cashier standing, rapid counter rush, direct lighting. | High-density grid, Numpad on right edge, instant visual depression feedback (heuristic: target <50ms; never exceed 100ms), one-tap tender buttons. |
| **Kitchen Display (KDS)** | 3-meter distance, steam, grease, poor viewing angles. | High-contrast dark theme, oversized typography, status-colored cards, hardware keyboard / bump-bar navigation. |
| **Manager Mobile** | On-the-go, one-handed thumb grip. | Primary actions inside the easy-reach lower-third thumb zone; destructive actions require explicit confirmation and must not sit next to frequent actions. |
| **Admin Web Console** | Desktop monitor, office environment, mouse & keyboard. | High data density, sortable tables, keyboard navigation shortcuts, multi-column filters. |

### 3. Spatial Restaurant & Retail Patterns
Alexander's architectural patterns that inform this section include **Street Cafe (88)**, **Food Stands (93)**, **Intimacy Gradient (127)**, **Eating Atmosphere (182)**, and **Cooking Layout (184)**. The following software patterns are derived for this skill, not Alexander pattern names:
- **Linear Checkout Flow (derived):** Fast, linear checkout (`Items -> Modifiers -> Subtotal -> Cash -> Receipt -> Drawer Kick`).
- **Expo Synchronization View (derived):** Dedicated expo screen synchronizing multi-station preparation (Hot Kitchen + Bar + Salad) before marking an order ready for pickup.
- **Blind Shift Drop (derived):** Forcing the cashier to input counted cash blind (without showing the system expected total) to prevent theft and calculate auditable cash variance.

---

## Architectural Decision Checklist (Before Writing Any Module)

When building or reviewing any module in an enterprise multi-stack project, run this checklist:

```markdown
### 1. Domain & Ledger
- [ ] Is the movement ledger the source of truth, with any stored balance a reconcilable projection updated in the same transaction?
- [ ] Is money stored exclusively as integer minor units with an ISO 4217 currency code (scale derived from the currency), with allocation for splits?
- [ ] Is the rounding policy documented and tested?
- [ ] Does this transaction ideally touch only a single Aggregate Root?
- [ ] Is external vendor data sanitized behind an Anti-Corruption Layer (ACL)?

### 2. Distributed Sync & Reliability
- [ ] Is the state change committed atomically with an Outbox event record?
- [ ] Does every retryable mutation API accept and enforce a unique idempotency key?
- [ ] Are retry mechanisms using capped exponential backoff with randomized jitter?
- [ ] Are failed asynchronous tasks routed to a Dead-Letter Queue (DLQ)?

### 3. Design System & Cross-Platform UI
- [ ] Are all colors, spaces, and typography derived from centralized Design Tokens?
- [ ] Are directional layouts built with CSS Logical Properties where supported for full RTL/LTR parity?
- [ ] Do POS/mobile touch targets meet the platform baseline (44pt iOS / 48dp Android) and at least 48×48 CSS px where practical, validated on hardware, with a documented separation rule (e.g., 8px)?
- [ ] Does the component handle the states this skill requires: Default, Focus-visible, Active/Pressed, Loading, Disabled; Hover where a pointer exists; plus Error and Empty where the component can fail or have no data?
```

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
