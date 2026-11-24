# Capacity, recovery and architecture evolution

Use for sizing, SLOs, restore plans, report isolation, tenant moves or adopting additional infrastructure. All numerical examples below must be labelled as inputs or arithmetic, not measurements. [Research](primary-sources.md).

## Define useful work before measuring throughput

Record workload, payload distribution, devices/tenants, offered load, transaction mix, resource limits, versions, budget and expected totals from an independent evaluator. Count durable accepted, duplicate, rejected, review and dependency-pending commands separately. Measure completed business effects as well as persisted protocol outcomes. Returning review for every command cannot establish healthy sales throughput.

At minimum report p50/p95/p99, error/timeout/retry counts, useful throughput, oldest pending age, foreground latency, report freshness, CPU/RAM/I/O/WAL growth, connection wait and disk/backup capacity. Include failed attempts in the denominator. Do not average percentiles across nodes. Use an arrival model that includes queueing/slowdowns; distinguish offered from completed rate and document coordinated-omission limitations.

Backlog drain approximation: `time = backlog / (useful_capacity - ongoing_arrivals)` when capacity exceeds arrivals. Include retried work and actual operation-to-write amplification separately. Storage estimate: rows × sampled row/index bytes × retention plus WAL, temporary/backfill, backup and headroom. A count of business operations is not a count of DB writes or HTTP requests.

## SLO and recovery boundaries

Define the user-visible event, good-event criterion, denominator, window and exclusions for each SLI. Measure local durable checkout independently from central API availability, sync delay and paper output. Label missing branch uploads as unknown, rather than describing their old central report as current. Map alerts to a response owner/runbook; error-budget consumption is more actionable than arbitrary CPU paging. [SRE](https://sre.google/workbook/alerting-on-slos/).

Separate four clocks: restore service, recover central data, catch up clients, and reconcile unresolved effects. RTO stops at a defined usable service state, not at PostgreSQL process startup. RPO is the loss interval of a named recovery chain. A local disk destroyed before upload/copy can lose unsent facts despite perfect cloud backups. A 60-minute recovery consumes more than a 99.9% time-based 30-day budget of 43.2 minutes; reconcile the targets explicitly.

Use physical/base PostgreSQL backup plus its required archived WAL chain for PITR; a logical dump is a different recovery mechanism. Rehearse in an isolated destination with key/config recovery, replay identities, constraints, RLS and independent totals. Prevent the restored worker from contacting external systems until the reconciliation plan allows it. [PITR](https://www.postgresql.org/docs/18/continuous-archiving.html).

For local POS, validate effective WAL/FULL, actual encrypted native engine, persistent operation identities and supported consistent backup. Record artifact/cipher/key parameters and independent destination age. App termination and actual power/storage loss are different tests. Background mobile scheduling is not a recovery guarantee. [SQLite durability](https://sqlite.org/pragma.html#pragma_synchronous), [backup API](https://sqlite.org/backup.html), [Drift encryption](https://drift.simonbinder.eu/platforms/encryption/).

## Reporting and projections

Start with transactional views/queries where affordable. Introduce an asynchronously updated read model when required by contention/freshness. Each projection needs a version, durable source position, idempotent processing, rebuild procedure and reconciliation totals. Rebuild into a shadow generation, catch up, validate and atomically switch readers. Do not reset an active cursor while retaining partial old totals.

Protect foreground sales/sync with bounded report concurrency, indexed pagination, export jobs and query deadlines. If adopting a replica, state acceptable staleness and authoritative fallback for dependent reads. A replica and a read model solve different problems. Metrics should use bounded labels; keep individual operation/device IDs in access-controlled logs/traces, not unbounded time-series labels.

## Measured escalation ladder

| Symptom | First investigation | Later option and added obligations |
| --- | --- | --- |
| Slow query | SQL/plan, projection, index, data skew | Read model/replica; freshness and failover |
| Worker lag | Claims, fairness, batch size, downstream budget | More workers; fencing and consumer deduplication |
| Local checkout slow | UI isolate, fsync/storage, transaction scope | Device/storage change; retain durability settings |
| Large active tables | Actual plans, maintenance, retention and restore | Partitioning; constraints, pruning, migration rehearsal |
| Tenant overwhelms shared placement | Quotas, query/report load and home sizing | Dedicated placement; fenced move and recovery |
| Need independently operated component | Workload/team/fault domain evidence | Service/broker; contracts, ownership and operational budget |

No fixed number of rows, engineers, vnodes or TB universally mandates a topology. A distributed platform profile does not prove that its largest tenant fits a single home. Preserve measured admissibility before optimizing cost.

## Whole-tenant move design

Keep organization identity and client business/retry identities stable; placement generation is separate. Create and reconcile the target while the source remains the sole authority. At cutover, quiesce/fence source writers and workers, capture the final delta, verify counts/totals/outcomes, then change trusted routing and open one target authority. The fence must be enforced at writes, not only by DNS/router cache. Drain old in-flight requests and reject stale placement generations.

Offline commands arriving through an old route retain their original identities and are routed/retried under the new authority; they must not post at both homes. Treat rollback after target writes as a new reconciled move, not as changing a pointer back to a stale source. Rehearse backup/PITR, jobs, credential scope, reporting/export and audit paths. This is a derived future protocol, not an automatic instruction to deploy multiple homes in G0.

## Plan output contract

For each work package supply: current rule/source, prerequisites, owning files/contracts, interfaces, algorithm and transaction boundary, concurrency/failure cases, independent acceptance evidence, recovery/compatibility, status and open decisions. Use the project's existing test IDs and milestones where possible. Keep project-specific adoption decisions in project documents/ADRs; reusable references explain how to evaluate the alternatives.
