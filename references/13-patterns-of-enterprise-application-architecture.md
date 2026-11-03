# Book 13: Patterns of Enterprise Application Architecture (PoEAA)
**Author:** Martin Fowler  
**Category:** Enterprise Architectural Patterns & Data Persistence  
**Relevance to SaaS & ERP:** Accounting Ledgers, Unit of Work, Money Pattern, Optimistic Concurrency.

---

## 1. Executive Summary & Core Thesis
Enterprise software is fundamentally about handling complex, persistent business data, executing concurrent operations safely, and structuring business logic around durable enterprise records. Martin Fowler catalogs the foundational software patterns that power enterprise resource planning, financial ledgers, and database-backed applications.

---

## 2. Core Architectural Patterns for ERP Systems

### A. The Money Pattern
* Floating-point arithmetic (`0.1 + 0.2 = 0.30000000000000004`) destroys financial accounting and audit credibility.
* **The Pattern:** Money must always be represented as an integer of the smallest currency unit (e.g., cents, piasters / قروش) accompanied by an ISO currency code.
  - *In Egypt:* 100.50 EGP is stored as `10050` minor units, `EGP`.
  - All rounding logic (Half-Even / Banker's Rounding) must be explicitly codified.

### B. Accounting & Ledger Patterns
* **Posting Rules & The Double-Entry Ledger:**
  - Every financial transaction generates balanced journal entries:
    $$\sum \text{Debits} = \sum \text{Credits}$$
  - Direct database updates (`UPDATE accounts SET balance = balance + 50`) are strictly forbidden in accounting.
  - Every state change is an immutable journal line linking to a concrete source document (Sale, Refund, Purchase Receipt, Payroll Run).

### C. Concurrency Control: Optimistic Offline Lock
* When multiple terminals (or a web admin and a cashier) attempt to edit the same record concurrently:
  - **The Pattern:** The table contains a `version` or `revision` column.
  - When committing an update:
    ```sql
    UPDATE catalog_items 
    SET price = 12000, version = version + 1 
    WHERE id = :id AND version = :expected_version;
    ```
  - If zero rows are affected, a concurrency collision occurred. The system rejects the silent overwrite and forces explicit resolution.

### D. The Unit of Work (نمط وحدة العمل)
* Maintains a list of objects affected by a business transaction and coordinates the writing out of changes and the resolution of concurrency problems in a single database transaction.

---

## 3. Architectural Rules Extracted for our Master Skill
1. **The Integer Money Invariant:** All currency and monetary calculations must be executed and stored in integer minor units with an explicit ISO currency attribute.
2. **The Double-Entry Invariance Rule:** Every posted journal entry must validate balance ($\Delta = 0$) inside the database transaction before commitment.
3. **Optimistic Versioning Rule:** All mutable operational aggregates must maintain a monotonic `revision` number to enforce optimistic concurrency control during concurrent updates and offline synchronization.
