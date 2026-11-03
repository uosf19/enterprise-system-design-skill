# Enterprise System Design & Multi-Stack Architecture Skill

A universal, production-grade architectural playbook for designing, auditing, and building mission-critical enterprise SaaS platforms, offline-first distributed applications, and cross-platform design systems.

Synthesized from **15 foundational books** in systems thinking, design systems, distributed data, and domain architecture.

---

## 📚 The 15 Foundational References

This skill distills the core principles, mathematical invariants, and architectural rules from:

1. **Thinking in Systems: A Primer** — *Donella H. Meadows*
2. **Atomic Design** — *Brad Frost*
3. **Design Systems: A practical guide to creating design languages** — *Alla Kholmatova*
4. **Design Systems Handbook** — *InVision (Marco Suarez et al.)*
5. **Expressive Design Systems** — *Yesenia Perez-Cruz*
6. **Laying the Foundations: How to design, build, and maintain a design system** — *Andrew Couldwell*
7. **Building Design Systems: Unify User Experiences** — *Sarrah Vesselov & Taurie Davis*
8. **Practical UI Patterns for Design Systems** — *Diana MacDonald*
9. **Frontend Architecture for Design Systems** — *Micah Godbolt*
10. **A Pattern Language: Towns, Buildings, Construction** — *Christopher Alexander et al.*
11. **Designing Data-Intensive Applications (DDIA)** — *Martin Kleppmann*
12. **Implementing Domain-Driven Design (IDDD)** — *Vaughn Vernon*
13. **Patterns of Enterprise Application Architecture (PoEAA)** — *Martin Fowler*
14. **Enterprise Integration Patterns (EIP)** — *Gregor Hohpe & Bobby Woolf*
15. **Designing for Touch** — *Josh Clark*

Detailed chapter-by-chapter extractions are available in the [`references/`](references/) directory.

---

## 🏛️ The Four Pillars

```text
       [ Universal Enterprise Architecture ]
                          |
   +--------------+---------------+--------------+----------------+
   |              |               |              |                |
[ PILLAR 1 ]   [ PILLAR 2 ]    [ PILLAR 3 ]   [ PILLAR 4 ]
Domain &       Distributed     Atomic Cross-  Ergonomics &
Systemic       Data & Sync     Platform       Spatial Pattern
Invariants     Engine          Design System  Language
(Systems + DDD) (DDIA + EIP)    (Tokens + UI)  (Touch + Context)
```

1. **Domain & Systemic Invariants:** Append-only ledgers, double-entry financial balancing, Integer Money pattern (no floats), Bounded Contexts, and Anti-Corruption Layers (ACL).
2. **Distributed Data & Resilient Sync:** Transactional Outbox pattern, compound Idempotency keys `(org_id, device_id, op_id)`, monotonic client sequencing, cursor-based pull, and Dead-Letter Queue (DLQ) quarantining.
3. **Atomic Cross-Platform Design System:** Single source-of-truth tokens (JSON) compiling to Tailwind/CSS and Flutter Dart, 4px/8px modular grid, CSS Logical Properties for bidirectional (RTL/LTR) parity, and strict vs. loose governance.
4. **Touch & Physical Ergonomics:** Minimum $48\text{px} \times 48\text{px}$ hit targets with 8px buffer margins, physical context adaptation (High-contrast KDS vs High-density POS), and spatial operational patterns.

---

## 📁 Repository Structure

```text
enterprise-system-design/
├── SKILL.md                 # Master skill playbook & execution rules
├── README.md                # Skill overview & catalog
├── references/              # Detailed extractions from all 15 books
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

## 🚀 How to Install & Use in Antigravity or AI Agent Workspaces

### Option A: Install as a Global Antigravity Skill
Clone or copy this repository into your global or project skills directory:
```bash
# In your Antigravity skills directory
git clone https://github.com/uosf19/enterprise-system-design-skill.git enterprise-system-design
```

### Option B: Use in Any Project
Reference `SKILL.md` in your project's architectural guidelines or system prompts. AI agents will automatically activate this skill when designing enterprise data models, offline sync pipelines, or design systems.

---

## 📄 License
MIT License. Created by Youssef Osman.
