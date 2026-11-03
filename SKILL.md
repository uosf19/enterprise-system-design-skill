---
name: enterprise-system-design
description: Enterprise multi-stack system architecture, offline-first data sync, domain-driven design, and cross-platform design systems distilled from 15 foundational architecture and design books. Use when designing, reviewing, or implementing complex enterprise SaaS architectures, offline-capable systems, financial/inventory ledgers, and multi-client frontend architectures.
---

# Enterprise System Design & Multi-Stack Architecture Skill

This skill is a universal, production-grade architectural playbook for designing, auditing, and scaling mission-critical SaaS platforms, offline-first distributed applications, and cross-platform design systems. It synthesizes foundational principles from 15 canonical engineering and design books into actionable invariants, design heuristics, and implementation checklists.

---

## The Four Architectural Pillars

```text
       [ Universal Enterprise Architecture ]
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
*Foundations: Thinking in Systems (Meadows), Implementing DDD (Vernon), Patterns of Enterprise App Architecture (Fowler).*

### 1. The Stock & Flow Ledger Invariant
- **Rule:** Never update a running balance or quantity directly (`UPDATE account SET balance = balance + x` is forbidden).
- **Implementation:** Balances are derived projections of an immutable, append-only ledger of historical movements:
  $$\text{Stock Balance}(t) = \sum \text{Inflows} - \sum \text{Outflows}$$
- **Double-Entry Balance:** Every financial journal entry must enforce debit-credit equality ($\sum \text{Debits} - \sum \text{Credits} = 0$) inside the database transaction before commitment.

### 2. The Money Pattern
- **Rule:** Floating-point arithmetic is strictly prohibited for monetary representation.
- **Implementation:** Store and calculate all money in integer minor units (e.g., piasters / cents) paired with an explicit ISO 4217 currency code:
  ```typescript
  interface Money {
    readonly amount: number; // Integer minor units (e.g., 15050 for 150.50 EGP)
    readonly currency: "EGP" | "USD" | "EUR";
  }
  ```
- Rounding must use deterministic Banker's Rounding (Half-Even) defined in the domain core.

### 3. Bounded Context & Aggregate Boundaries
- **Rule:** An Aggregate is the sole consistency boundary. A database transaction may modify exactly **one** Aggregate instance.
- **Cross-Aggregate Updates:** Must be orchestrated asynchronously via Domain Events and the Transactional Outbox.
- **Anti-Corruption Layer (ACL):** External vendor payloads (payment gateways, tax authorities, biometric clocks) must be parsed and mapped through an Anti-Corruption Layer before entering the domain model.

---

## Pillar 2: Distributed Data & Resilient Sync Engine
*Foundations: Designing Data-Intensive Applications (Kleppmann), Enterprise Integration Patterns (Hohpe).*

### 1. The Transactional Outbox Pattern
- **Rule:** Domain state mutations and their outbound event records must be committed atomically inside the exact same ACID database transaction:
  ```sql
  -- Atomic Domain Write + Outbox Event
  BEGIN;
  INSERT INTO sales_orders (...) VALUES (...);
  INSERT INTO outbox_events (id, event_type, payload, status) 
  VALUES (gen_random_uuid(), 'sales.order_placed.v1', :payload, 'pending');
  COMMIT;
  ```
- A background outbox worker polls or streams events from the outbox table to the message broker.

### 2. Deterministic Idempotency Key Invariant
- **Rule:** All write APIs, sync endpoints, and external webhook receivers must enforce idempotency using compound unique constraints:
  $$\text{Idempotency Key} = (\text{organization\_id}, \text{device\_id}, \text{operation\_id})$$
- Retrying an identical operation must return the cached original result without re-executing business logic or creating duplicate records.

### 3. Monotonic Logical Sequencing & Conflict Resolution
- **Rule:** Never rely on device wall-clock timestamps for distributed transaction ordering.
- **Implementation:**
  - Clients emit a strictly monotonic sequence counter per device (`local_sequence`).
  - Servers validate the sequence and track cloud changes with opaque sequential cursors.
  - Conflict resolution follows semantic rules: append-only ledgers merge; editable entity conflicts are isolated into an explicit operator review queue (`AttendanceCase`, `ReconciliationException`), never resolved via silent Last-Write-Wins (LWW).

### 4. Dead-Letter Queue (DLQ) & Quarantine Isolation
- External adapter failures (e.g., rejected tax submissions, malformed webhooks) must be isolated into a Dead Letter Queue after configured exponential backoff retries ($2^n + \text{jitter}$). Never drop messages or halt the upstream processing queue.

---

## Pillar 3: Atomic Cross-Platform Design Systems
*Foundations: Atomic Design (Frost), Frontend Architecture (Godbolt), Design Systems Handbook (InVision), Laying the Foundations (Couldwell).*

### 1. The Platform-Agnostic Token Architecture
- Author all design decisions in a single JSON/YAML token registry (`packages/design-tokens`):
  - **Global Tokens:** Primitive palette (`color.palette.emerald.500: #10b981`).
  - **Semantic Tokens:** Purpose-bound (`color.status.positive: {color.palette.emerald.500}`).
  - **Component Tokens:** Component-scoped (`badge.background.success: {color.status.positive}`).
- Automated CI pipeline (Style Dictionary) compiles tokens into:
  - `tailwind.config.js` and CSS Custom Properties for Web & Desktop (Next.js & Tauri).
  - `AppTheme.dart` constants for Mobile (Flutter).

### 2. The 4px/8px Spatial Rhythm & Bidirectional (RTL/LTR) Rules
- **Grid Rule:** All paddings, margins, widths, and heights must be multiples of 4px or 8px. Arbitrary pixel values are rejected by linters.
- **Directional Agnosticism:** All CSS styling must use **CSS Logical Properties**:
  - Use `margin-inline-start` instead of `margin-left`.
  - Use `padding-inline-end` instead of `padding-right`.
  - Use `inset-inline-start` instead of `left`.

### 3. Dual-Tier System Governance
- **Strict Governance:** Mission-critical transactional interfaces (POS checkout, billing, payments, tax compliance) enforce strict component usage; no inline CSS or untokenized overrides permitted.
- **Loose Governance:** Reporting, analytics dashboards, and internal exploration allow composable layout variation while honoring base tokens.

---

## 4. Pillar 4: Ergonomics & Spatial Pattern Language
*Foundations: A Pattern Language (Alexander), Designing for Touch (Clark), Expressive Design Systems (Perez-Cruz).*

### 1. Touch Target Geometry & Spacing
- **The $48\text{px} \times 48\text{px}$ Invariant:** Any interactive target on a touchscreen device (POS, KDS, Mobile) must have a minimum physical bounding box of $48\text{px} \times 48\text{px}$.
- **Safety Buffer:** Adjacent touchable targets must maintain a minimum physical margin of 8px to prevent accidental mis-taps during peak operating hours.

### 2. Environmental Context Adaptation

| Surface | Operating Environment | Architectural Requirements |
| :--- | :--- | :--- |
| **POS Terminal** | Cashier standing, rapid counter rush, direct lighting. | High-density grid, Numpad on right edge, instant visual depression feedback ($<50\text{ms}$), one-tap tender buttons. |
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
- [ ] Is money stored exclusively as integer minor units with an ISO currency code?
- [ ] Does this transaction touch only a single Aggregate Root?
- [ ] Is external vendor data sanitized behind an Anti-Corruption Layer (ACL)?

### 2. Distributed Sync & Reliability
- [ ] Is the state change committed atomically with an Outbox event record?
- [ ] Does every mutation API accept and enforce a unique compound Idempotency Key?
- [ ] Are retry mechanisms using exponential backoff with randomized jitter?
- [ ] Are failed asynchronous tasks routed to a Dead-Letter Queue (DLQ)?

### 3. Design System & Cross-Platform UI
- [ ] Are all colors, spaces, and typography derived from centralized Design Tokens?
- [ ] Are layouts built with CSS Logical Properties (`inline`/`block`) for full RTL/LTR parity?
- [ ] Are all touch targets on POS/Mobile at least 48px x 48px with 8px margins?
- [ ] Does the component handle all 5 states (Default, Hover, Active, Loading, Disabled)?
```

---

## Detailed References Directory
Deep dive into individual book summaries and mathematical foundations in the `references/` directory:
- [01. Thinking in Systems (Donella Meadows)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/01-thinking-in-systems.md)
- [02. Atomic Design (Brad Frost)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/02-atomic-design.md)
- [03. Design Systems (Alla Kholmatova)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/03-design-systems-kholmatova.md)
- [04. Design Systems Handbook (InVision)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/04-design-systems-handbook.md)
- [05. Expressive Design Systems (Yesenia Perez-Cruz)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/05-expressive-design-systems.md)
- [06. Laying the Foundations (Andrew Couldwell)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/06-laying-the-foundations.md)
- [07. Building Design Systems (Sarrah Vesselov)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/07-building-design-systems.md)
- [08. Practical UI Patterns (Diana MacDonald)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/08-practical-ui-patterns.md)
- [09. Frontend Architecture (Micah Godbolt)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/09-frontend-architecture.md)
- [10. A Pattern Language (Christopher Alexander)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/10-a-pattern-language.md)
- [11. Designing Data-Intensive Applications (Martin Kleppmann)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/11-designing-data-intensive-applications.md)
- [12. Implementing Domain-Driven Design (Vaughn Vernon)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/12-implementing-domain-driven-design.md)
- [13. Patterns of Enterprise Application Architecture (Martin Fowler)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/13-patterns-of-enterprise-application-architecture.md)
- [14. Enterprise Integration Patterns (Gregor Hohpe)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/14-enterprise-integration-patterns.md)
- [15. Designing for Touch (Josh Clark)](file:///c:/Users/yusse/Documents/Projects/SaaS/docs/skills/enterprise-system-design/references/15-designing-for-touch.md)
