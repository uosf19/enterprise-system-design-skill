---
name: enterprise-system-design
description: Enterprise system design and multi-stack architecture for complex SaaS platforms, offline-first restaurant/retail POS systems, financial and inventory ledgers, domain-driven design, and cross-platform design systems. Synthesized from 15 foundational books. Use when designing, reviewing, or implementing enterprise SaaS, POS/retail, offline sync, RTL interfaces, design tokens, or multi-client frontend architecture.
---

# Enterprise System Design & Multi-Stack Architecture Skill

This skill is a production-grade architectural playbook for designing, auditing, and scaling mission-critical SaaS platforms. Its primary focus is offline-first restaurant/retail POS systems and cross-platform design systems, but its invariants apply to other multi-tenant enterprise domains. It synthesizes 15 foundational engineering and design books into actionable invariants, heuristics, and checklists.

---

## Scope & Assumptions

- **Primary domain:** multi-tenant, offline-first restaurant/retail SaaS (POS, KDS, manager mobile, admin web).
- **Stack examples:** Next.js, Tauri, Flutter, and PostgreSQL are illustrative; the patterns are stack-agnostic.
- **Regulatory examples:** ETA, Paymob, Fawry, and similar references are illustrative, not legal or tax advice. Verify current rules for each jurisdiction and document type.
- **Not a complexity mandate:** simple CRUD or low-scale applications should not adopt DDD aggregates, outbox, or event sourcing without a real consistency, audit, or scale requirement.
- **Language note:** Arabic glosses in the references support MENA teams; they are explanatory, not required terminology.

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

### 1. The Stock & Flow Ledger Invariant
- **Rule:** Never update a running balance or quantity directly (`UPDATE account SET balance = balance + x` is forbidden).
- **Implementation:** Balances are derived projections of an immutable, append-only ledger of historical movements:
  ```text
  Stock Balance(t) = Σ Inflows − Σ Outflows
  ```
- **Double-Entry Balance:** Every financial journal entry must enforce debit-credit equality (`Σ Debits = Σ Credits`, equivalently `Σ Debits − Σ Credits = 0`) inside the database transaction before commitment.

### 2. The Money Pattern
- **Rule:** Floating-point arithmetic is strictly prohibited for monetary representation.
- **Implementation:** Store and calculate all money in integer minor units (e.g., piasters/cents) paired with an explicit ISO 4217 currency code and its minor-unit scale:
  ```typescript
  interface Money {
    readonly amount: number;   // integer minor units (15050 = 150.50 EGP)
    readonly currency: string; // ISO 4217 code: "EGP", "USD", "JPY", "KWD"
    readonly scale: number;    // decimal digits: 2 for EGP/USD, 0 for JPY, 3 for KWD
  }
  ```
- `number` is safe for typical amounts; for very large ledgers or high-precision aggregation, use `bigint` or a decimal library.
- **Rounding:** Use a documented deterministic policy. Banker's rounding (half-even) is a safe default for aggregated financial calculations; some jurisdictions require half-up. Encode and test the policy once in the domain core.

### 3. Bounded Context & Aggregate Boundaries
- **Rule:** An Aggregate is the consistency boundary. Prefer one Aggregate instance per database transaction; when multiple aggregates must change together, coordinate them asynchronously via Domain Events and the Transactional Outbox, or explicitly justify the larger transaction.
- **Anti-Corruption Layer (ACL):** External vendor payloads (payment gateways, tax authorities, biometric clocks) must be parsed and mapped through an Anti-Corruption Layer before entering the domain model.

---

## Pillar 2: Distributed Data & Resilient Sync Engine
*Foundations: Designing Data-Intensive Applications (Kleppmann), Enterprise Integration Patterns (Hohpe).*

### 1. The Transactional Outbox Pattern
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

### 3. Monotonic Logical Sequencing & Conflict Resolution
- **Rule:** Never rely on device wall-clock timestamps for distributed transaction ordering.
- **Implementation:**
  - Clients emit a strictly monotonic sequence counter per device (`local_sequence`).
  - Servers validate the sequence and track cloud changes with opaque sequential cursors.
  - Append-only ledgers converge via idempotent replay and set union.
  - Editable entity conflicts affecting money, stock, or compliance are isolated into an explicit operator review queue (`AttendanceCase`, `ReconciliationException`), never resolved via silent Last-Write-Wins (LWW). Reserve LWW for non-critical fields with a documented classification.

### 4. Dead-Letter Queue (DLQ) & Quarantine Isolation
- External adapter failures (e.g., rejected tax submissions, malformed webhooks) must be isolated into a Dead Letter Queue after capped exponential backoff with jitter (`min(max_backoff, 2^n + jitter)`, e.g., a 5-minute cap). Never drop messages or halt the upstream processing queue.

---

## Pillar 3: Atomic Cross-Platform Design Systems
*Foundations: Atomic Design (Frost), Frontend Architecture (Godbolt), Design Systems Handbook (InVision), Laying the Foundations (Couldwell).*

### 1. The Platform-Agnostic Token Architecture
- Author all design decisions in a single JSON/YAML token registry (`packages/design-tokens`):
  - **Global Tokens:** Primitive palette (`color.palette.emerald.500: #10b981`).
  - **Semantic Tokens:** Purpose-bound (`color.status.positive: {color.palette.emerald.500}`).
  - **Component Tokens:** Component-scoped (`badge.background.success: {color.status.positive}`).
- Automated CI pipeline (Style Dictionary or equivalent) compiles tokens into:
  - Tailwind configuration or CSS Custom Properties for Web & Desktop (Next.js & Tauri).
  - Flutter theme extensions or constants for Mobile (Flutter).

### 2. The 4px/8px Spatial Rhythm & Bidirectional (RTL/LTR) Rules
- **Grid Rule:** Layout spacing and component dimensions must use the 4px/8px scale. Hairline borders (1px), focus rings (2px), and optical corrections are documented exceptions; linters should allow them explicitly.
- **Directional Agnosticism:** Use **CSS Logical Properties** for directional layout where supported:
  - Use `margin-inline-start` instead of `margin-left`.
  - Use `padding-inline-end` instead of `padding-right`.
  - Use `inset-inline-start` instead of `left`.
  - Properties without logical equivalents (e.g., some transforms and background positions) need direction-aware helpers.

### 3. Dual-Tier System Governance
- **Strict Governance:** Mission-critical transactional interfaces (POS checkout, billing, payments, tax compliance) enforce strict component usage; no inline CSS or untokenized overrides permitted.
- **Loose Governance:** Reporting, analytics dashboards, and internal exploration allow composable layout variation while honoring base tokens.

---

## Pillar 4: Ergonomics & Spatial Pattern Language
*Foundations: A Pattern Language (Alexander), Designing for Touch (Clark), Expressive Design Systems (Perez-Cruz).*

### 1. Touch Target Geometry & Spacing
- **The 48×48 Baseline:** Interactive targets on touchscreen devices (POS, KDS, Mobile) should be at least 48×48 CSS px (48dp on Android; the iOS HIG minimum is 44pt). Validate the physical size on target hardware; CSS px is not a fixed physical unit.
- **Safety Buffer:** Adjacent touchable targets must maintain at least 8px separation to prevent accidental mis-taps during peak operating hours.

### 2. Environmental Context Adaptation

| Surface | Operating Environment | Architectural Requirements |
| :--- | :--- | :--- |
| **POS Terminal** | Cashier standing, rapid counter rush, direct lighting. | High-density grid, Numpad on right edge, instant visual depression feedback (target <50ms; never exceed 100ms), one-tap tender buttons. |
| **Kitchen Display (KDS)** | 3-meter distance, steam, grease, poor viewing angles. | High-contrast dark theme, oversized typography, status-colored cards, hardware keyboard / bump-bar navigation. |
| **Manager Mobile** | On-the-go, one-handed thumb grip. | Primary actions inside the natural lower-third thumb zone; destructive actions quarantined in top corners. |
| **Admin Web Console** | Desktop monitor, office environment, mouse & keyboard. | High data density, sortable tables, keyboard navigation shortcuts, multi-column filters. |

### 3. Spatial Restaurant & Retail Patterns
- **The Counter Pattern:** Fast, linear checkout flow (`Items -> Modifiers -> Subtotal -> Cash -> Receipt -> Drawer Kick`).
- **The Kitchen Pass Pattern:** Dedicated Expo screen synchronizing multi-station preparation (Hot Kitchen + Bar + Salad) before marking an order as ready for pickup.
- **The Blind Shift Drop Pattern:** Forcing the cashier to input counted cash blind (without showing system expected total) to prevent theft and calculate auditable cash variance.

---

## Architectural Decision Checklist (Before Writing Any Module)

When building or reviewing any module in an enterprise multi-stack project, run this checklist:

```markdown
### 1. Domain & Ledger
- [ ] Are balances and quantities derived from an append-only movement ledger?
- [ ] Is money stored exclusively as integer minor units with an ISO 4217 currency code and scale?
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
- [ ] Are all touch targets on POS/Mobile at least 48px × 48px with 8px separation?
- [ ] Does the component handle all six interaction states (Default, Hover, Focus-visible, Active, Loading, Disabled)?
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
