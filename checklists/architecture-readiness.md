# Architecture Readiness Checklist

Use the sections relevant to the change. Start with the project's approved operating rules and current phase. For POS durability and shared UI, use the linked [sync](offline-sync-resilience.md) and [design](design-system-tokens.md) checks.

---

## 1. Domain Modeling & DDD Discipline
- [ ] **Ubiquitous Language:** Are all class names, method names, and database column names using domain terms rather than generic technical terms (e.g., `ticket.bump()` vs `updateTicketStatus(2)`)?
- [ ] **Transaction Boundary:** Are immediate invariants and all records required to protect them committed together? Are effects that tolerate delay separated through an outbox where needed?
- [ ] **Value Object Immutability:** Are money, units of measure, coordinates, and date ranges modeled as immutable Value Objects?
- [ ] **Money:** Are posted amounts exact integer minor units with currency? Verify exact intermediate arithmetic, overflow and wire encoding, policy-specific rounding stages and signed ties, and allocations that conserve the total.
- [ ] **Ledger and Projection:** For stock or cash, can balances be rebuilt from posted facts? Do updates that enforce immediate limits serialize concurrent writes and commit the movement with its projection?
- [ ] **Append-Only Auditing:** Are state changes that affect money, stock, permissions, or fiscal receipts append-only with explicit reversal records?

---

## 2. Multi-Tenancy & Security
- [ ] **Organization ID Enforcement:** Is tenant scope derived from trusted identity and enforced in application access and persistence? For shared PostgreSQL with RLS, test the actual API and worker roles with cross-tenant reads and writes, including connection reuse.
- [ ] **Tenant-Safe References:** Do scoped foreign keys/natural unique namespaces prevent a row from referencing another organization's data? Verify missing context, rollback/savepoint recovery, worker scope, restricted functions/views and separate migration authority. See [data design](../references/16-production-data-and-tenancy.md).
- [ ] **Role & Capability Matrix:** Are user permissions enforced based on explicit capabilities rather than hardcoded `if (role === 'admin')` checks?
- [ ] **Approval Boundaries:** Where project policy requires separate approval, verify that the initiator cannot self-approve. Test scope separately for branch, warehouse, device and worker.
- [ ] **Secret Hygiene:** Are tokens, API keys, and certificates isolated in environment secret stores, never committed to git or stored in SQLite unencrypted?

---

## 3. Observability & Operational Readiness
- [ ] **Correlation Tracking:** Does every incoming request carry or generate a `request_id`, `correlation_id`, and `tenant_id` propagated through all async jobs and logs?
- [ ] **Structured Logging:** Are logs emitted as structured JSON without leaking PII, raw passwords, or credit card numbers?
- [ ] **Health & Readiness Endpoints:** Do readiness checks reflect the dependencies actually needed to serve traffic, separately from process health?
- [ ] **Alerting & SLOs:** Are critical flows covered by SLOs with actionable alerts (error rate, latency, outbox lag, DLQ depth)?

---

## 4. Integration & API Contracts
- [ ] **API Versioning:** Are public APIs versioned with a documented backward-compatibility policy and deprecation window?
- [ ] **Error Taxonomy:** Do APIs return stable machine-readable error codes and actionable messages without leaking internals?
- [ ] **Webhook Security:** Are inbound webhooks signature-verified, replay-protected, and idempotent?
- [ ] **ACL & Contract Tests:** Are external adapters isolated, with versioned contracts and stable business IDs? Keep synthetic, provider-test and live evidence separately identified. Verify signed external calls as required by the provider contract.
- [ ] **Delivery and Consumers:** For deferred effects, verify atomic outbox persistence, retry, duplicate delivery, and ordering where business rules require it. Start with the project's database worker. Adopt a broker only on evidence of a delivery need.
- [ ] **Claim and Cursor Correctness:** Check stale-worker generation guards, atomic consumer inbox/effects, snapshot/delta continuity, expired-cursor recovery and preservation of pending device facts. See [worker and sync protocols](../references/17-sync-cursors-and-workers.md).
- [ ] **Dependency Timeouts:** Are timeouts, retries, and circuit breakers configured per dependency rather than globally?

---

## 5. Testing & Delivery
- [ ] **Invariant Tests:** Use unit and property tests for money, units and allocation. Share exact fixtures and encoding across languages that implement the same rule, including edge values, rounding and stable hashes.
- [ ] **Real Database Tests:** Test concurrent deduplication, transaction rollback and scoped access using real persistence and application roles. Add contract fixtures for supported older clients and failure drills for restart, restore, printing and lost responses where applicable.
- [ ] **Migration Safety:** Are schema migrations reversible or forward-fixable and tested against a production-like snapshot?
- [ ] **Delivery Checks:** Does the chosen local or CI workflow run the relevant automated checks and record hardware or live evidence separately?
- [ ] **Rollback Plan:** Is there a rollback or feature-flag plan for risky changes?

---

## 6. Performance & Capacity
- [ ] **Latency Budgets:** Measure checkout, sync and recovery on a named device and workload before adopting a budget. For offline checkout, ensure network or printer availability cannot prevent durable local commit. Preserve the workload for repeat measurements.
- [ ] **Query Health:** Are slow queries and missing indexes caught before production?
- [ ] **Backpressure:** Are queue depth, outbox lag, and DLQ size monitored, with backpressure or load-shedding behavior defined?
- [ ] **Peak Headroom:** Is there capacity headroom for peak load and reconnect storms?
- [ ] **Useful Work and Placement:** Separate offered/completed operations, durable outcomes, posted effects and retries; include independent expected totals and reporting contention. Verify the largest tenant within its single write home, rather than relying on platform totals. See [capacity/evolution](../references/18-capacity-recovery-and-evolution.md).

---

## 7. Resilience & Recovery
- [ ] **Backups & PITR:** Are backups and point-in-time recovery tested, with defined RTO/RPO?
- [ ] **Replay & Reconciliation:** Is there a documented DLQ replay and reconciliation procedure?
- [ ] **Graceful Degradation:** Does the system degrade predictably when dependencies are unavailable?
- [ ] **DR Drills:** Are disaster-recovery drills performed and reviewed?
- [ ] **Recovery Boundaries:** Record central recovery, device loss interval, catch-up and reconciliation separately. A backup schedule is not a measured RPO. Define safe outbox/provider behavior after restore, and write fencing/reconciliation before a future tenant routing switch.

---

## 8. ERP Core Concerns (review prompts; add detail per project)
- [ ] **Period Close:** Where accounting is in scope, define period locking and linked corrections under the approved accounting policy. Preserve the original record.
- [ ] **Document Numbering:** Are invoice/receipt numbers generated per the legal rules of the jurisdiction (verify whether gaps are allowed), with a plan for offline issuance?
- [ ] **Inventory Costing:** Is the costing method (FIFO, weighted average, other) chosen, documented, and tested?
- [ ] **Tenancy Model:** Is the isolation approach (shared schema with RLS, schema-per-tenant, database-per-tenant) chosen with its trade-offs recorded?
- [ ] **Authorization Model:** Is access control (RBAC/ABAC, capabilities) defined centrally and tested?
- [ ] **Reporting Separation:** Do heavy reports run on read models or replicas instead of the transactional tables?
- [ ] **Effective-Dated Rules:** Are tax and price rules stored with effective dates, and is the applied rule snapshotted on each document?
