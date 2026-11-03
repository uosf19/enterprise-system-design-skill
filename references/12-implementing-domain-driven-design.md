# Book 12: Implementing Domain-Driven Design (IDDD)
**Author:** Vaughn Vernon (building on Eric Evans)  
**Category:** Strategic & Tactical Software Architecture  
**Relevance to SaaS & ERP:** Bounded Contexts, Aggregates, Invariants, Clean Hexagonal Architecture.

---

## 1. Executive Summary & Core Thesis
Complex enterprise software projects fail when the codebase devolves into an unstructured "Big Ball of Mud" where business logic leaks across database queries, controllers, and UI components. Domain-Driven Design provides the strategic tools to partition the domain into autonomous **Bounded Contexts** and the tactical tools (Entities, Value Objects, Aggregates, Repositories) to enforce business rules and invariants inside isolated boundaries.

---

## 2. Strategic Design: Partitioning the ERP Domain

```text
[ Core Enterprise Monolith ]
   |
   +---> Bounded Context: Organization & Tenancy
   +---> Bounded Context: Catalog & Menu Revision
   +---> Bounded Context: Sales & Order Processing
   +---> Bounded Context: Kitchen & Fulfillment
   +---> Bounded Context: Inventory & Recipe Consumption
   +---> Bounded Context: Workforce & Attendance Evidence
   +---> Bounded Context: Finance & Accounting Subledger
```

* **The Ubiquitous Language (اللغة الموحدة):** Every term in the code must strictly reflect the language of business experts. If the chef says "Ticket Line Fired", the domain model must have `ticketLine.fire()`, not `updateTicketStatus(3)`.
* **Context Mapping & Anti-Corruption Layers (ACL):** When integrating with external systems (e.g., Egyptian Tax Authority ETA, Paymob, or vendor biometric terminals), the core domain must never import external data models. An **Anti-Corruption Layer (Adapter)** translates external vendor payloads into clean internal domain value objects.

---

## 3. Tactical Design: Enforcing Invariants

### A. Entities vs. Value Objects
* **Entity:** An object defined by its enduring unique identity across time (e.g., `SalesOrder`, `Employee`, `Branch`). Two orders with identical items are different if their IDs differ.
* **Value Object:** An immutable object defined entirely by its attributes, with no conceptual identity (e.g., `Money(amount: 1500, currency: "EGP")`, `Address`, `RecipeYield`).
  - *Rule:* Favor Value Objects whenever possible to eliminate mutability bugs.

### B. Aggregates & Aggregate Roots (جذور التجميع)
* An Aggregate is a cluster of associated entities and value objects treated as a single unit for data changes.
* Every Aggregate has exactly **one Aggregate Root** (e.g., `SalesOrder` is the root; `OrderLine` and `Discount` are internal children).
* **The Aggregate Invariant Rule:** External code may only hold references to the Aggregate Root. Any modification to an internal child must go through a method on the Aggregate Root to ensure business invariants are never bypassed.

```typescript
// Correct DDD: Aggregate Root enforces invariants
class SalesOrder {
  private lines: OrderLine[] = [];
  private isPaid: boolean = false;

  public addLine(item: CatalogItem, qty: number): void {
    if (this.isPaid) {
      throw new DomainError("Cannot modify a paid order");
    }
    this.lines.push(new OrderLine(item, qty));
    this.recalculateTotals();
  }
}
```

---

## 4. Architectural Rules Extracted for our Master Skill
1. **The Aggregate Boundary Invariant:** A database transaction should ideally modify only one Aggregate instance at a time. Modifications to other aggregates should be handled via Domain Events and asynchronous Outbox processing.
2. **Value Object Primacy for Money & Quantity:** Never represent monetary values or physical quantities as raw numbers (`number` or `float`). Always encapsulate them in strongly typed Value Objects with explicit currency and unit of measure.
3. **The Anti-Corruption Gate (ACL):** External vendor payloads (ETA receipts, payment webhooks, biometric clocks) must be sanitized and converted through an Anti-Corruption Layer before entering the domain core.
