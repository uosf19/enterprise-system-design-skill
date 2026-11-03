# Book 07: Building Design Systems
**Authors:** Sarrah Vesselov & Taurie Davis  
**Category:** Engineering Workflows & Organizational Alignment  
**Relevance to SaaS & ERP:** Monorepo UI Integration, GitLab Case Study, Proving Business ROI.

---

## 1. Executive Summary & Core Thesis
Drawing from their real-world experience building the GitLab enterprise design system (Pajamas), Vesselov and Davis bridge the cultural and technical gap between product design and engineering. A design system must be treated as an internal product with measurable business return: decreasing delivery cycle times, reducing visual/technical debt, and democratizing cross-team contributions.

---

## 2. Key Frameworks & Methodologies

### A. The Business Case & ROI Metrics
To sustain a design system, you must measure and report its impact:
* **Velocity Metric:** Time required to deliver a new enterprise feature (e.g., measuring cycle time before vs. after standardized component adoption).
* **Defect Metric:** Reduction in UI-related customer bug reports and accessibility tickets.
* **Consistency Score:** Percentage of production screens composed strictly of certified system components.

### B. Engineering Collaboration & Code Architecture
* **Single Source of Truth in Code:**
  - Component code must be self-documenting with TypeScript/JSDoc types.
  - Component props should have strict default values and validate edge cases (empty states, loading states, error boundaries).
* **Automated Visual Regression:**
  - Continuous integration pipelines must run automated visual diffs (e.g., Storybook test runner or Playwright snapshots) on every pull request to catch accidental CSS/layout breaks.

### C. The Federated Contribution Model
* Avoid the "Gatekeeper Bottleneck":
  - Provide a clear template for proposing, building, and contributing new components.
  - Require automated tests, accessibility audits, and documentation before merging.

---

## 3. Architectural Rules Extracted for our Master Skill
1. **Self-Documenting Component APIs:** Every UI component must expose fully typed props with explicit TypeScript/Dart interfaces and fallback defaults.
2. **Automated Visual CI Gates:** Pull requests modifying shared UI components must pass automated visual regression snapshots across desktop, tablet, and mobile viewport sizes.
3. **The State Completeness Invariant:** Every interactive component must explicitly handle and display all 5 states: `Default`, `Hover/Focus`, `Active/Pressed`, `Loading`, and `Disabled`.
