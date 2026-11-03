# Architecture Readiness Checklist

Use this checklist prior to approving any module design, pull request, or architectural RFC in an enterprise SaaS system.

---

## 1. Domain Modeling & DDD Discipline
- [ ] **Ubiquitous Language:** Are all class names, method names, and database column names using domain terms rather than generic technical terms (e.g., `ticket.bump()` vs `updateTicketStatus(2)`)?
- [ ] **Aggregate Isolation:** Does each command handler or transaction modify only a single Aggregate Root?
- [ ] **Value Object Immutability:** Are money, units of measure, coordinates, and date ranges modeled as immutable Value Objects?
- [ ] **Zero Float Policy:** Is all monetary math executed in integer minor units (e.g., piasters / cents)? Are rounding strategies deterministic?
- [ ] **Append-Only Auditing:** Are state changes that affect money, stock, permissions, or fiscal receipts append-only with explicit reversal records?

---

## 2. Multi-Tenancy & Security
- [ ] **Organization ID Enforcement:** Does every database query and row-level security (RLS) policy validate `organization_id` at the database engine level?
- [ ] **Role & Capability Matrix:** Are user permissions enforced based on explicit capabilities rather than hardcoded `if (role === 'admin')` checks?
- [ ] **Maker-Checker Separation:** Are high-risk actions (approving cash variance, modifying past attendance, issuing refunds above threshold) protected from self-approval?
- [ ] **Secret Hygiene:** Are tokens, API keys, and certificates isolated in environment secret stores, never committed to git or stored in SQLite unencrypted?

---

## 3. Observability & Operational Readiness
- [ ] **Correlation Tracking:** Does every incoming request carry or generate a `request_id`, `correlation_id`, and `tenant_id` propagated through all async jobs and logs?
- [ ] **Structured Logging:** Are logs emitted as structured JSON without leaking PII, raw passwords, or credit card numbers?
- [ ] **Health & Readiness Endpoints:** Does the service expose separate `/healthz` (liveness) and `/readyz` (dependency check: DB, Cache) probes?
