# Cross-Platform Design System & Tokens Checklist

Use this checklist when creating, updating, or reviewing design tokens, UI components, and cross-platform frontend code.

---

## 1. Design Token Architecture & Discipline
- [ ] **Single Source of Truth:** Are all color, typography, elevation, and spacing values defined in the platform-agnostic token registry (`packages/design-tokens`), never hardcoded in component files?
- [ ] **Three-Tier Token Structure:** Do tokens follow the Global $\rightarrow$ Semantic $\rightarrow$ Component hierarchy?
- [ ] **Automated Asset Compilation:** Does updating the token registry automatically regenerate Tailwind configs, CSS variables, and Flutter Dart classes?
- [ ] **4px/8px Grid Conformance:** Are all layout paddings, margins, and component dimensions exact multiples of 4px or 8px?

---

## 2. Bidirectional & Multilingual Support (RTL / LTR)
- [ ] **CSS Logical Properties:** Does all CSS use `inline-start` / `inline-end` instead of `left` / `right`?
- [ ] **Optical Baseline Alignment:** Do Arabic (e.g., Cairo / Noto Kufi) and Latin typography scales align on the same baseline grid to eliminate layout shifts when switching language?
- [ ] **Mirroring Compliance:** Are directional icons (back arrows, disclosure chevrons, stepper indicators) automatically mirrored in RTL mode?

---

## 3. Touch & Physical Ergonomics
- [ ] **Minimum Hit Target:** Are all clickable and touchable targets on POS, KDS, and Mobile screens at least $48\text{px} \times 48\text{px}$?
- [ ] **8px Buffer Rule:** Is there at least an 8px physical buffer between adjacent touchable buttons to prevent mis-taps during rush hours?
- [ ] **Instant Touch Feedback:** Does pressing a touch element provide immediate visual depression/highlight state in $<50\text{ms}$?
- [ ] **Five-State Completeness:** Does the component implement and visually handle all 5 interaction states: `Default`, `Hover/Focus`, `Active/Pressed`, `Loading`, and `Disabled`?
