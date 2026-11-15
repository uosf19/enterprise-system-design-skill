# Book 08: Practical UI Patterns for Design Systems
**Author:** Diana MacDonald  
**Category:** Interaction Design Patterns & User Flows  
**Relevance to SaaS & ERP:** Complex F&B Interaction Patterns, Anti-Pattern Elimination, Flow Orchestration.

---

## 1. Executive Summary & Core Thesis
UI patterns are proven, recurring solutions to common interaction design problems. Components do not exist in isolation; their true value is unlocked when they are orchestrated into cohesive, end-to-end user journeys. Diana MacDonald provides a practical framework for dissecting patterns, evaluating them against business and technical constraints, and eliminating destructive anti-patterns.

---

## 2. Core Methodologies & Pattern Taxonomy
*Source note: the book's seven chapters are Introducing UI Patterns; Tap into Patterns; Deciding Which Pattern to Use and When; Patterns in Design Systems; Anti-Patterns and Dark Patterns; Mixing and Matching Patterns; and Conclusion. It does not present a named "Anatomical Evaluation" framework. The three-part evaluation below is derived for this skill.*

### A. Deconstructing a Pattern: Context, Constraints, and Trade-offs
Before adopting or building any UI pattern, evaluate it against three derived pillars:
1. **User Mental Model:** Does this pattern match the user's natural expectations from daily life?
2. **Technical Feasibility:** Can this pattern operate smoothly offline, on low-end hardware, and across varied network latencies?
3. **Business Implications:** Does this pattern prevent user error, speed up transactional throughput, or increase data accuracy?

### B. High-Value Interaction Patterns for Enterprise ERP & POS
*Derived for this skill: the book teaches pattern selection and composition; these restaurant/POS patterns are domain applications, not patterns from the book.*

| Pattern | Problem Solved | Implementation in the Reference Architecture |
| :--- | :--- | :--- |
| **Split-Bill Orchestrator** | Splitting a restaurant table check by seat, item, or percentage without calculation errors. | Interactive check allocation drawer with visual balance indicator (Total − Allocated = 0). |
| **Modifier Progression Flow** | Selecting complex recipe modifications (Size → Temperature → Milk Type → Syrups) quickly during a rush. | Single/Multi-select stepper with mandatory minimums, visual pricing deltas, and keyboard bypass. |
| **Blind shift count (optional)** | An approved workflow calls for a count before disclosing the expected amount. | Capture the authorized count and present variance according to the drawer policy. |
| **Exception Resolution Queue** | Reviewing biometric/WhatsApp attendance discrepancies without altering raw evidence. | Two-column audit view: Raw evidence on left, calculated timesheet on right, reason selector in center. |

### C. Identifying and Eliminating Anti-Patterns
*The anti-pattern category is from the book; the three named labels below are this skill's shorthand.*
* **The "Accidental Click" Anti-Pattern:** Placing irreversible actions (e.g., "Void Entire Bill" or "Refund Cash") right next to frequent actions (e.g., "Print Receipt") without a deliberate confirmation or pin-code gate.
* **The "Silent Failure" Anti-Pattern:** Allowing an offline action to appear successful while silently dropping it from the queue without visual status indicators.
* **The "Data Amnesia" Anti-Pattern:** Clearing a partially filled order or complex form if a minor network glitch or modal dismissal occurs.

---

## 3. Architectural Rules for This Skill
1. **The Irreversible Action Gate:** Any destructive, financial, or data-modifying action (Void, Refund, Delete) must require secondary confirmation or elevated authorization.
2. **State Persistence on Disruption:** Define which drafts may be stored on each client. Test durability and recovery for supported offline actions. Keep draft, committed sale, sync and printing states explicit.
3. **Explicit Error Remediation:** Error notifications must never be purely informational; they must always include a concrete corrective action (e.g., "Retry Sync", "Switch Printer", "Call Supervisor").
