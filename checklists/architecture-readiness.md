# Architecture Readiness Checklist

Use this checklist prior to approving any module design, pull request, or architectural RFC in an enterprise SaaS system.

---

## 1. Domain Modeling & DDD Discipline
- [ ] **Ubiquitous Language:** Are all class names, method names, and database column names using domain terms rather than generic technical terms (e.g., `ticket.bump()` vs `updateTicketStatus(2)`)?
- [ ] **Aggregate Isolation:** Does each command handler or transaction ideally modify only a single Aggregate Root, with cross-aggregate changes coordinated via domain events?
- [ ] **Value Object Immutability:** Are money, units of measure, coordinates, and date ranges modeled as immutable Value Objects?
- [ ] **Zero Float Policy:** Is all monetary math executed in integer minor units with an ISO 4217 currency code and minor-unit scale? Is the rounding policy documented and tested?
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
- [ ] **Alerting & SLOs:** Are critical flows covered by SLOs with actionable alerts (error rate, latency, outbox lag, DLQ depth)?

---

## 4. Integration & API Contracts
- [ ] **API Versioning:** Are public APIs versioned with a documented backward-compatibility policy and deprecation window?
- [ ] **Error Taxonomy:** Do APIs return stable machine-readable error codes and actionable messages without leaking internals?
- [ ] **Webhook Security:** Are inbound webhooks signature-verified, replay-protected, and idempotent?
- [ ] **ACL & Contract Tests:** Are external adapters isolated behind an Anti-Corruption Layer and covered by contract tests?
- [ ] **Dependency Timeouts:** Are timeouts, retries, and circuit breakers configured per dependency rather than globally?

---

## 5. Testing & Delivery
- [ ] **Invariant Tests:** Are domain invariants covered by unit tests, with property-based tests where the input space is large?
- [ ] **Real Database Tests:** Do integration tests exercise real constraints (unique indexes, RLS, transactions) rather than mocks alone?
- [ ] **Migration Safety:** Are schema migrations reversible or forward-fixable and tested against a production-like snapshot?
- [ ] **CI Gates:** Do CI gates block on failing tests, type/lint errors, and visual regressions?
- [ ] **Rollback Plan:** Is there a rollback or feature-flag plan for risky changes?

---

## 6. Performance & Capacity
- [ ] **Latency Budgets:** Are p95/p99 budgets defined for critical flows (checkout, sync, login)?
- [ ] **Query Health:** Are slow queries and missing indexes caught before production?
- [ ] **Backpressure:** Are queue depth, outbox lag, and DLQ size monitored, with backpressure or load-shedding behavior defined?
- [ ] **Peak Headroom:** Is there capacity headroom for peak load and reconnect storms?

---

## 7. Resilience & Recovery
- [ ] **Backups & PITR:** Are backups and point-in-time recovery tested, with defined RTO/RPO?
- [ ] **Replay & Reconciliation:** Is there a documented DLQ replay and reconciliation procedure?
- [ ] **Graceful Degradation:** Does the system degrade predictably when dependencies are unavailable?
- [ ] **DR Drills:** Are disaster-recovery drills performed and reviewed?
