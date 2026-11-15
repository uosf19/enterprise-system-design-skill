# Enterprise System Design & Multi-Stack Architecture Skill

A playbook for offline-first restaurant and retail POS, enterprise data boundaries, and shared design systems.

The study notes draw on 15 book references and related material. The [source evidence index](references/source-evidence.md) records the material checked, including excerpts and summaries, and distinguishes derived implementation guidance.

> **Scope note:** the primary domain is multi-tenant, offline-first restaurant/retail SaaS (POS, KDS, manager mobile, admin web). The invariants are adaptable to other enterprise domains. Regulatory references are illustrative, not legal or tax advice.

---

## 📚 The 15 Foundational References

This skill distills the core principles, invariants, and architectural rules from:

1. **Thinking in Systems: A Primer** — *Donella H. Meadows*
2. **Atomic Design** — *Brad Frost*
3. **Design Systems: A Practical Guide to Creating Design Languages** — *Alla Kholmatova*
4. **Design Systems Handbook** — *Marco Suarez, Jina Anne, Katie Sylor-Miller, Diana Mounter, and Roy Stanfield (InVision)*
5. **Expressive Design Systems** — *Yesenia Perez-Cruz*
6. **Laying the Foundations: A Book About Design Systems** — *Andrew Couldwell*
7. **Building Design Systems: Unify User Experiences through a Shared Design Language** — *Sarrah Vesselov & Taurie Davis*
8. **Practical UI Patterns for Design Systems** — *Diana MacDonald*
9. **Frontend Architecture for Design Systems** — *Micah Godbolt*
10. **A Pattern Language: Towns, Buildings, Construction** — *Christopher Alexander et al.*
11. **Designing Data-Intensive Applications (DDIA)** — *Martin Kleppmann*
12. **Implementing Domain-Driven Design (IDDD)** — *Vaughn Vernon*
13. **Patterns of Enterprise Application Architecture (PoEAA)** — *Martin Fowler*
14. **Enterprise Integration Patterns (EIP)** — *Gregor Hohpe & Bobby Woolf*
15. **Designing for Touch** — *Josh Clark*

Detailed summaries and extracted architectural rules are available in the [`references/`](references/) directory.

---

## Source Notes & Derived Guidance

The skill distinguishes three kinds of material:

- **Book frameworks and named patterns** are attributed to their source books.
- **External standards and platform guidance** (ISO 4217, WCAG, SemVer, Apple HIG, Material Design) are cited as standards.
- **Derived guidance** includes the POS workflows, component lifecycle, token tiers, state coverage, and physical-device heuristics. Use approved project decisions and the applicable release gates when implementing them.

When a section mixes book content with synthesis, the derived material is labeled in place. See `SKILL.md` for the full attribution policy.

---

## 🏛️ The Four Pillars

```text
       [ Enterprise Reference Architecture ]
                          |
   +--------------+---------------+--------------+----------------+
   |              |               |              |                |
[ PILLAR 1 ]   [ PILLAR 2 ]    [ PILLAR 3 ]   [ PILLAR 4 ]
Domain &       Distributed     Atomic Cross-  Ergonomics &
Systemic       Data & Sync     Platform       Spatial Pattern
Invariants     Engine          Design System  Language
(Systems + DDD) (DDIA + EIP)    (Tokens + UI)  (Touch + Context)
```

1. **Domain & Systemic Invariants:** Append-only ledgers and double-entry balancing (domain standards), Fowler's Money pattern with integer minor units as this skill's implementation standard, Bounded Contexts, and Anti-Corruption Layers (ACL).
2. **Distributed Data & Resilient Sync:** Transactional outbox, epoch-scoped POS retry keys, stable business identities across restores, cursor-based pull and distinct retry, review and verification paths.
3. **Atomic Cross-Platform Design System:** Single source-of-truth tokens (JSON) compiling to Tailwind/CSS and Flutter theme assets, a 4px/8px modular grid (skill standard), CSS Logical Properties for bidirectional (RTL/LTR) parity, and strictness calibrated across surfaces.
4. **Touch & Physical Ergonomics:** Platform hit-target baselines (44pt iOS / 48dp Android), a 48×48 CSS px POS/KDS target heuristic, Material's 8dp spacing guidance adapted to 8px, physical context adaptation (high-contrast KDS vs. high-density POS), and spatial operational patterns.

---

## 🧭 How to Apply This Skill

1. **Classify the task:** domain/ledger, offline sync, design system, touch ergonomics, or architecture review.
2. **Read progressively:** start with [`SKILL.md`](SKILL.md), then open only the relevant reference(s) and checklist.
3. **Make decisions explicit:** state the pattern, trade-offs, and unresolved risks.
4. **Verify:** run the relevant checklist before approving a design, RFC, or pull request.

For the full routing guide, see [How to Apply This Skill](SKILL.md#how-to-apply-this-skill).

---

## 📁 Repository Structure

```text
enterprise-system-design-skill/
├── SKILL.md                 # Master skill playbook, scope, workflow & invariants
├── README.md                # Skill overview & catalog
├── LICENSE                  # MIT license
├── references/              # Detailed summaries from all 15 books
│   ├── 01-thinking-in-systems.md
│   ├── 02-atomic-design.md
│   ├── 03-design-systems-kholmatova.md
│   ├── 04-design-systems-handbook.md
│   ├── 05-expressive-design-systems.md
│   ├── 06-laying-the-foundations.md
│   ├── 07-building-design-systems.md
│   ├── 08-practical-ui-patterns.md
│   ├── 09-frontend-architecture.md
│   ├── 10-a-pattern-language.md
│   ├── 11-designing-data-intensive-applications.md
│   ├── 12-implementing-domain-driven-design.md
│   ├── 13-patterns-of-enterprise-application-architecture.md
│   ├── 14-enterprise-integration-patterns.md
│   ├── 15-designing-for-touch.md
│   ├── offline-pos-workflow.md
│   └── source-evidence.md
└── checklists/              # Operational checklists for architecture & code review
    ├── architecture-readiness.md
    ├── offline-sync-resilience.md
    └── design-system-tokens.md
```

---

## Installation

Install the complete `enterprise-system-design` folder from the chosen release using your agent's documented skills directory. For a project-scoped Codex installation, place it under `.agents/skills/` and verify it appears in the skill selector. Use one discoverable copy per name. You can also point an agent directly to [SKILL.md](SKILL.md) without installing it. [Codex skill discovery](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills)

---

## 🧰 What's Inside

- [`SKILL.md`](SKILL.md) — scope, workflow, four pillars, and checklist routing.
- [Offline POS workflow](references/offline-pos-workflow.md) — transaction boundaries and failure scenarios.
- [`checklists/architecture-readiness.md`](checklists/architecture-readiness.md) — domain, security, observability, integration, testing, performance, recovery gates, and ERP core review prompts.
- [`checklists/offline-sync-resilience.md`](checklists/offline-sync-resilience.md) — edge durability, sync protocol, conflicts, schema migration, and observability.
- [`checklists/design-system-tokens.md`](checklists/design-system-tokens.md) — tokens, RTL/LTR, touch ergonomics, accessibility, theming, and lifecycle.

---

## 📄 License

MIT License. Created by Youssef Osman ([@uosf19](https://github.com/uosf19)). See [LICENSE](LICENSE).

---

## Disclaimer

These are independent study notes and architectural syntheses. They are not affiliated with, endorsed by, or a substitute for the original books; book titles and trademarks belong to their respective owners. Regulatory references (for example, ETA) are illustrative and do not constitute legal or tax advice. Verify current rules for your jurisdiction and document type.
