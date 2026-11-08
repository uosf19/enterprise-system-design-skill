# Cross-Platform Design System & Tokens Checklist

Use this checklist when creating, updating, or reviewing design tokens, UI components, and cross-platform frontend code.

Items labeled "skill standard" or "heuristic" are implementation choices, not claims from the 15 source books.

---

## 1. Design Token Architecture & Discipline
- [ ] **Single Source of Truth:** Are all color, typography, elevation, and spacing values defined in the platform-agnostic token registry (`packages/design-tokens`), never hardcoded in component files?
- [ ] **Three-Tier Token Structure (skill implementation model):** Do tokens follow the Global → Semantic → Component hierarchy, or a documented equivalent?
- [ ] **Automated Asset Compilation:** Does updating the token registry automatically regenerate Tailwind/CSS variables and Flutter theme extensions or constants?
- [ ] **4px/8px Grid Conformance (skill standard):** Are layout paddings, margins, and component dimensions on the 4px/8px scale, with documented exceptions for 1px borders, 2px focus rings, and optical corrections?

---

## 2. Bidirectional & Multilingual Support (RTL / LTR)
- [ ] **CSS Logical Properties:** Does directional CSS use `inline-start` / `inline-end` instead of `left` / `right` where supported, with direction-aware helpers for properties that have no logical equivalent?
- [ ] **Optical Baseline Alignment:** Do Arabic (e.g., Cairo / Noto Kufi) and Latin typography scales align on the same baseline grid to eliminate layout shifts when switching language?
- [ ] **Mirroring Compliance:** Are directional icons (back arrows, disclosure chevrons, stepper indicators) automatically mirrored in RTL mode?

---

## 3. Touch & Physical Ergonomics
- [ ] **Platform Hit Target:** Do POS, KDS, and mobile targets meet the platform baseline (44pt iOS / 48dp Android) and at least 48×48 CSS px where practical, validated on target hardware?
- [ ] **Touch Separation:** Is there at least an 8px buffer between adjacent touchable buttons (adapting Material's 8dp guidance) to reduce mis-taps during rush hours?
- [ ] **Instant Touch Feedback (heuristic):** Does pressing a touch element provide immediate visual depression/highlight state (target <50ms; never exceed 100ms, validated on the device)?
- [ ] **State Completeness (skill requirement):** Does the component handle `Default`, `Focus-visible`, `Active/Pressed`, `Loading`, and `Disabled`; `Hover` only where a pointer exists; and `Error`/`Empty` where it can fail or have no data?

---

## 4. Accessibility, Theming & Lifecycle
- [ ] **Contrast:** Do color tokens meet WCAG 2.2 AA contrast in every theme (light, dark, high-contrast)?
- [ ] **Focus Visibility:** Is `focus-visible` styling tokenized and clearly visible on every interactive component?
- [ ] **Theme Derivation:** Are light, dark, and high-contrast themes generated from the same semantic tokens rather than forked palettes?
- [ ] **Token Versioning:** Are token and component versions and deprecations documented with migration paths?
- [ ] **Accessible States:** Do form controls define Selected and Read-only states in addition to the states above? Do touch targets also satisfy WCAG 2.2 Target Size (Minimum)?
