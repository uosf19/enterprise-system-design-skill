# Book 13: Patterns of Enterprise Application Architecture (PoEAA)
**Author:** Martin Fowler  
**Category:** Enterprise Architectural Patterns & Data Persistence  
**Relevance to SaaS & ERP:** Money Pattern, Unit of Work, Optimistic Offline Lock; double-entry ledger as a domain extension.

---

## 1. Executive Summary & Core Thesis
Enterprise software is fundamentally about handling complex, persistent business data, executing concurrent operations safely, and structuring business logic around durable enterprise records. Martin Fowler catalogs the foundational software patterns that power enterprise resource planning, financial ledgers, and database-backed applications.

---

## 2. Core Architectural Patterns for ERP Systems
*Source note: the PoEAA patterns used in this section are Money, Unit of Work, and Optimistic Offline Lock. Double-entry accounting and integer minor-unit storage are domain/implementation standards adopted for this skill, not patterns from PoEAA.*

### A. The Money Pattern
* Fowler's Money pattern represents a monetary value as an **amount plus a currency**, and makes rounding explicit. It does **not** require integer minor units or prescribe a specific storage format.
* The pattern also covers **allocation**: dividing an amount without losing or inventing minor units (100.00 split three ways gives 33.34, 33.33, 33.33). This is needed for split bills, discounts, and tax distribution.
* Floating-point arithmetic (`0.1 + 0.2 = 0.30000000000000004`) is unsuitable for monetary calculations; use integer minor units or a decimal type.
* *Implementation standard for this skill:* store and calculate money in integer minor units (e.g., cents, piasters / قروش) with an ISO 4217 currency code; the scale comes from a currency table rather than being stored per value.
  - *In Egypt:* 100.50 EGP is stored as `10050` minor units, `EGP`.
  - All rounding logic (Half-Even / Banker's Rounding) must be explicitly codified and tested.

### B. Domain Extension: Double-Entry Ledger (Not a PoEAA Pattern)
* PoEAA's catalog does not include a double-entry accounting pattern. Double-entry is an accounting domain standard adopted for this skill.
* **Posting Rules & The Double-Entry Ledger:**
  - Every financial transaction generates balanced journal entries:
    Σ Debits = Σ Credits
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

## 3. Architectural Rules for This Skill
1. **The Integer Money Standard (skill implementation):** All currency and monetary calculations must be executed and stored in integer minor units with an explicit ISO currency attribute, unless a documented decimal representation is required.
2. **The Double-Entry Ledger Standard (domain extension):** Every posted journal entry must validate balance (Δ = 0) inside the database transaction before commitment.
3. **The Optimistic Offline Lock Rule (PoEAA pattern):** All mutable operational aggregates must maintain a monotonic `revision` number to enforce optimistic concurrency control during concurrent updates and offline synchronization.
