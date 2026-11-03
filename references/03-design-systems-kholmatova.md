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

### B. Shared Ubiquitous Language (اللغة المشتركة)
* A design system must align the **user mental model**, the **design model**, and the **code implementation**:
  - If the business calls a completed kitchen item "Bumped", the code must call it `bumped`, the UI button must say "Bump", and the analytics event must be `kitchen.ticket_item_bumped`.
  - Avoid presentational code naming (`red-text`, `card-left`). Use semantic domain naming (`fiscal-warning`, `primary-action`, `station-pass`).

### C. System Architecture: Strict vs. Loose Governance

| Dimension | Strict Design Systems (صارمة) | Loose Design Systems (مرنة) |
| :--- | :--- | :--- |
| **Philosophy** | "One right way to do things" | "Guiding guardrails with room to innovate" |
| **Rule Enforcement** | Enforced via automated linters and strict approval gates | Encouraged via guidelines and recommended defaults |
| **Best Used For** | Mission-critical financial, compliance, and POS screens where errors cause cash loss | Exploratory dashboards, marketing pages, internal experimentation |
| **ERP Application** | **POS, Billing, Tax & Accounting:** 100% Strict. | **Reports & Analytics:** Configurable & Loose. |

---

## 3. Product-Driven Principles (The 3 to 5 Invariant Rules)
Kholmatova stresses that design principles must be **opinionated and actionable**—helping engineers resolve real-world trade-offs:
1. **Speed Over Decoration in Transactional Screens:** The cashier must complete a transaction in under 3 taps. Beauty never trumps ergonomics.
2. **Deterministic Clarity in Financial States:** Every financial status (Paid, Partial, Refunded, Voided) must have an unambiguous, standardized visual marker.
3. **No Dead Ends:** Every error state must provide an actionable remedy or fallback route.

---

## 4. Architectural Rules for This Skill
1. **The Semantic Naming Invariant:** Never name a token, component, or state based on its visual representation. Always name it according to its semantic business purpose.
2. **Dual-Tier Governance:** Enforce strict compilation rules on core transactional modules (POS, Billing, Tax) while allowing flexible layout composition on reporting surfaces.
3. **Living Documentation Integration:** A pattern is only considered "Production-Ready" when its code, design tokens, and domain rules are documented together in a single accessible registry.
