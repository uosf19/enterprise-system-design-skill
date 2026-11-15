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
* *Implementation guidance for this skill:* store posted money in integer minor units with a currency code. Keep intermediate values exact using integer ratios or an explicit decimal policy. Validate ranges across database, arithmetic and wire encoding.
  - *In Egypt:* 100.50 EGP is stored as `10050` minor units, `EGP`.
  - Specify each rounding point, signed ties and deterministic remainder order from the applicable policy. A rounding mode used in a synthetic fixture does not establish a provider or fiscal rule.

### B. Domain Extension: Double-Entry Ledger (Not a PoEAA Pattern)
* PoEAA's catalog does not include a double-entry accounting pattern. Double-entry is an accounting domain standard adopted for this skill.
* **Posting Rules & The Double-Entry Ledger:**
  - Every financial transaction generates balanced journal entries:
    Σ Debits = Σ Credits
  - Record the source journal entry. Any stored balance is a reconcilable projection and may be updated in the same posting transaction.
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
  - If zero rows are affected, distinguish a stale version from an absent or out-of-scope row without exposing unauthorized data. Return the appropriate safe outcome.

### D. The Unit of Work (نمط وحدة العمل)
* Maintains a list of objects affected by a business transaction and coordinates the writing out of changes and the resolution of concurrency problems in a single database transaction.

---

## 3. Architectural Rules for This Skill
1. **Exact money (skill implementation):** Compare currency before arithmetic. Protect intermediate precision and bounds. Test signed, tie, limit and allocation cases across every client that calculates them.
2. **The Double-Entry Ledger Standard (domain extension):** Every posted journal entry must validate balance (Δ = 0) inside the database transaction before commitment.
3. **Optimistic Offline Lock (PoEAA pattern):** Use a checked revision for editable records where optimistic concurrency fits. Choose separate protections for offline ownership, financial posting, and stock receipt reconciliation according to their invariants.
