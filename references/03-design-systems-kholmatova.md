# Book 03: Design Systems: A Practical Guide
**Author:** Alla Kholmatova  
**Category:** Design Language & Systems Strategy  
**Relevance to SaaS & ERP:** Functional vs. Perceptual Patterns, Shared Language, Governance Models.

---

## 1. Executive Summary & Core Thesis
A design system is far more than a UI component kit or Figma file; it is an interconnected ecosystem of **tangible patterns** (code, design tokens, UI components, documentation) and **intangible shared practices** (brand values, terminology, design principles, team workflows). A system must be engineered around the **underlying purpose of the specific product**, not generic beauty.

---

## 2. Key Frameworks & Models

### A. Functional Patterns vs. Perceptual Patterns
* **Functional Patterns (الأنماط الوظيفية):**
  - Structural, interactive components that solve specific user actions (e.g., buttons, data tables, modals, tabs, forms).
  - *Core Guideline:* Name and design functional patterns by the **behaviors they enable**, not by what they look like (e.g., `ActionTrigger` instead of `BigBlueButton`).
* **Perceptual Patterns (الأنماط الإدراكية):**
  - The emotional and brand aesthetic: typography hierarchies, color scales, spatial rhythms, iconography style, and voice/tone.
  - *Core Guideline:* Perceptual patterns act as the unifying glue that ensures different functional elements feel like they belong to the same product family.

### B. Shared Language (اللغة المشتركة)
* Kholmatova's term is **shared language** or **design language**. "Ubiquitous language" is DDD terminology, used here only as a cross-reference to the same alignment problem.
* A design system must align the **user mental model**, the **design model**, and the **code implementation**:
  - If the business calls a completed kitchen item "Bumped", the code must call it `bumped`, the UI button must say "Bump", and the analytics event must be `kitchen.ticket_item_bumped`.
  - Avoid presentational code naming (`red-text`, `card-left`). Use semantic domain naming (`fiscal-warning`, `primary-action`, `station-pass`).

### C. System Architecture: Strictness, Modularity, and Ownership
Kholmatova treats **strictness** as a system parameter on a spectrum, not as a binary choice. She also discusses **modularity** (how many independent, interchangeable parts the system has) and **centralized vs. distributed ownership** (who maintains and evolves it). A system can be strict about core transactional patterns while remaining loose about exploratory surfaces.

| Parameter | Stricter end | Looser end | Derived ERP application |
| :--- | :--- | :--- | :--- |
| **Strictness** | Enforced rules and limited variation | Guidelines and recommended defaults | POS, billing, and tax screens: stricter; reports and analytics: looser |
| **Modularity** | Few, tightly integrated components | Many small, composable components | Shared primitives centralized; domain screens compose them |
| **Ownership** | Central design-system team | Distributed product teams | Central core plus federated contributions with review gates |

*Source note: the POS/reporting split is a derived application for this skill, not a prescription from Kholmatova.*

---

## 3. Design Principles (The 3 to 5 Guidance)
Kholmatova recommends a small set of design principles—roughly **three to five**—that are opinionated and actionable. The examples below are derived for this skill:
1. **Efficient checkout (derived):** Measure time and tap errors for a named scenario on the target device. Simplify frequent actions while preserving the approval and confirmation steps the workflow requires.
2. **Deterministic Clarity in Financial States:** Every financial status (Paid, Partial, Refunded, Voided) must have an unambiguous, standardized visual marker.
3. **No Dead Ends:** Every error state must provide an actionable remedy or fallback route.

---

## 4. Architectural Rules for This Skill
1. **Semantic naming:** Use purpose-based names for semantic tokens and states. Primitive palette values can use descriptive color names.
2. **Calibrated Governance:** Treat strictness as a spectrum. Apply tighter enforcement to core transactional modules (POS, Billing, Tax) and looser guidance to reporting surfaces; document where each applies.
3. **Living Documentation Integration:** A pattern is only considered "Production-Ready" when its code, design tokens, and domain rules are documented together in a single accessible registry.
