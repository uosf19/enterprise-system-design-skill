# Enterprise System Design & Multi-Stack Architecture Skill

A production-grade architectural playbook for designing, auditing, and building mission-critical SaaS platforms, with a primary focus on offline-first restaurant/retail POS systems and cross-platform design systems.

Synthesized from **15 foundational books** in systems thinking, design systems, distributed data, and domain architecture.

> **Scope note:** the primary domain is multi-tenant, offline-first restaurant/retail SaaS (POS, KDS, manager mobile, admin web). The invariants are adaptable to other enterprise domains. Regulatory references are illustrative, not legal or tax advice.

---

## 📚 The 15 Foundational References

This skill distills the core principles, invariants, and architectural rules from:

1. **Thinking in Systems: A Primer** — *Donella H. Meadows*
2. **Atomic Design** — *Brad Frost*
3. **Design Systems: A Practical Guide to Creating Design Languages** — *Alla Kholmatova*
4. **Design Systems Handbook** — *InVision (Marco Suarez et al.)*
5. **Expressive Design Systems** — *Yesenia Perez-Cruz*
6. **Laying the Foundations: How to Design, Build, and Maintain a Design System** — *Andrew Couldwell*
7. **Building Design Systems: Unify User Experiences** — *Sarrah Vesselov & Taurie Davis*
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

1. **Domain & Systemic Invariants:** Append-only ledgers, double-entry financial balancing, integer Money pattern (no floats), Bounded Contexts, and Anti-Corruption Layers (ACL).
2. **Distributed Data & Resilient Sync:** Transactional Outbox pattern, compound idempotency keys `(org_id, device_id, op_id)`, monotonic client sequencing, cursor-based pull, and Dead-Letter Queue (DLQ) quarantining.
3. **Atomic Cross-Platform Design System:** Single source-of-truth tokens (JSON) compiling to Tailwind/CSS and Flutter theme assets, 4px/8px modular grid, CSS Logical Properties for bidirectional (RTL/LTR) parity, and strict vs. loose governance.
4. **Touch & Physical Ergonomics:** Minimum 48px × 48px hit targets with 8px separation, physical context adaptation (high-contrast KDS vs. high-density POS), and spatial operational patterns.

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
│   └── 15-designing-for-touch.md
└── checklists/              # Operational checklists for architecture & code review
    ├── architecture-readiness.md
    ├── offline-sync-resilience.md
    └── design-system-tokens.md
```

---

## Installation

### Option A: Install in an Agent Skills directory

Clone the repository into the skills directory used by your agent. The folder name should match the skill name (`enterprise-system-design`).

**Personal (available across projects):**

```bash
# Claude Code
git clone https://github.com/uosf19/enterprise-system-design-skill.git ~/.claude/skills/enterprise-system-design

# Cursor / Codex
git clone https://github.com/uosf19/enterprise-system-design-skill.git ~/.cursor/skills/enterprise-system-design
```

**Project-local (shared with the repository):**

```bash
git clone https://github.com/uosf19/enterprise-system-design-skill.git .claude/skills/enterprise-system-design

# Also commonly used: .cursor/skills/, .opencode/skills/
```

For Antigravity or another agent, use its documented global or project skills directory.

### Option B: Use in any project

Reference [`SKILL.md`](SKILL.md) in your project's architectural guidelines or system prompt. Agents that support Agent Skills can discover it through the frontmatter description; other agents can be pointed to `SKILL.md` explicitly.

---

## 🧰 What's Inside

- [`SKILL.md`](SKILL.md) — master playbook: scope, workflow, four pillars, and pre-flight checklist.
- [`references/`](references/) — 15 book summaries with extracted architectural rules.
- [`checklists/architecture-readiness.md`](checklists/architecture-readiness.md) — domain, security, observability, integration, testing, performance, and recovery gates.
- [`checklists/offline-sync-resilience.md`](checklists/offline-sync-resilience.md) — edge durability, sync protocol, conflicts, schema migration, and observability.
- [`checklists/design-system-tokens.md`](checklists/design-system-tokens.md) — tokens, RTL/LTR, touch ergonomics, accessibility, theming, and lifecycle.

---

## 📄 License

MIT License. Created by Youssef Osman ([@uosf19](https://github.com/uosf19)). See [LICENSE](LICENSE).

---

## Disclaimer

These are independent study notes and architectural syntheses. They are not affiliated with, endorsed by, or a substitute for the original books; book titles and trademarks belong to their respective owners. Regulatory references (for example, ETA) are illustrative and do not constitute legal or tax advice. Verify current rules for your jurisdiction and document type.
