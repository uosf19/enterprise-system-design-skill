# Production data boundaries, tenancy and caching

Read for a multi-tenant data design, competing financial mutations, query performance, cache adoption or schema changes. These are derived implementation recipes. Project policies and measured limits take precedence. [Evidence register](primary-sources.md#primary-source-register).

## Tenant scope is a transaction input

Resolve organization, actor, permitted branch/warehouse and enrolled device from authenticated authority. Bind that scope to the database unit of work; reject absent scope before issuing business queries. User-supplied organization IDs and job payloads are claims to validate.

For PostgreSQL use non-owner runtime roles without superuser/BYPASSRLS and a separate migration identity. RLS is defense against missing predicates, not protection against an application process that can freely impersonate tenants. Review raw SQL, views, functions, support access, exports and queues under the same boundary. [RLS](https://www.postgresql.org/docs/18/ddl-rowsecurity.html).

```sql
-- Illustrative policy; apply via reviewed migrations to every scoped table.
ALTER TABLE sales ENABLE ROW LEVEL SECURITY;
CREATE POLICY organization_scope ON sales
  USING (organization_id =
    nullif(current_setting('app.organization_id', true), '')::uuid)
  WITH CHECK (organization_id =
    nullif(current_setting('app.organization_id', true), '')::uuid);

BEGIN;
-- Parameter value comes from validated authority, not an arbitrary header.
SELECT set_config('app.organization_id', :trusted_organization_id, true);
-- All scoped queries use this connection and transaction.
COMMIT;
```

`USING` and `WITH CHECK` cover existing versus new row values. Missing context must deny access; malformed context fails closed. Use FORCE RLS where an owner is intentionally constrained; it never constrains a superuser/BYPASSRLS role. [Policy semantics](https://www.postgresql.org/docs/18/sql-createpolicy.html).

Transaction-local settings end at commit/rollback and may disappear on rollback to an earlier savepoint. Rebind safely before continuing. In .NET, use scoped DbContexts and bounded reusable Npgsql data sources; preserve driver reset. Do not retain mutable tenant state in singleton/background services. [SET](https://www.postgresql.org/docs/18/sql-set.html), [Npgsql](https://www.npgsql.org/doc/basic-usage.html).

## Relational integrity includes organization identity

Define tenant-owned references against `(organization_id, id)`, backed by a matching unique key. A sale line's foreign key to an item must include organization ID; a globally unique item ID alone does not establish that it belongs to the sale's tenant. Branch/drawer references need the additional same-scope relation where the domain requires it. Tenant RLS does not substitute for these constraints. Scope natural unique keys, such as external references, to their actual business namespace. Do not automatically tenant-scope truly global reference data. [Constraints](https://www.postgresql.org/docs/18/ddl-constraints.html).

## Financial transaction recipe

1. Validate shape and authority. Resolve the immutable retry identity and fingerprint.
2. Atomically arbitrate that identity under database uniqueness. A matching recorded command returns its original outcome; conflicting content takes review without posting again.
3. Serialize the affected authoritative aggregate, re-read its state, then check remaining refundable amounts, reservation or transition conditions. Two different operation IDs still compete on the same balance.
4. Commit allowed domain changes, append-only movements, projections protecting immediate invariants, command outcome and outbox together. Follow a documented lock order.
5. On deadlock/serialization failure, retry the whole unit with the same identity and bounded policy. On an uncertain commit, recover by persisted identity. Keep external calls outside retried transactions.

For an absent-row invariant, use a unique constraint or lock an existing parent; a row lock on no rows proves nothing. Example: one active shift per organization/drawer can use a partial unique index over active rows. Check the actual shift state vocabulary first. [Locking](https://www.postgresql.org/docs/18/explicit-locking.html), [transaction retries](https://www.postgresql.org/docs/18/transaction-iso.html), [EF uncertain commit](https://learn.microsoft.com/en-us/ef/core/miscellaneous/connection-resiliency).

## Query and data-shape decisions

Inventory actual reads: organization/branch filters, ordering, projected columns, pagination and frequency. Evaluate candidate indexes on representative data and the runtime role, accounting for write/WAL/storage costs. Keep keyset ordering deterministic with a tie-breaker. Do not use a hard row-count threshold as a sharding rule.

PostgreSQL heap/index storage differs from InnoDB clustered storage; a covering index does not eliminate all visibility checks. Large JSON/text is not automatically wrong: select only needed columns, consider update rate, concurrency, audit/recovery and transfer costs. Choose relational documents, file storage or object storage from those needs. [Index-only scans](https://www.postgresql.org/docs/18/indexes-index-only-scans.html), [TOAST](https://www.postgresql.org/docs/18/storage-toast.html).

## Cache adoption and read-after-write

Introduce a cache for measured disposable reads, not as the authority for money, authorization or stock. A key includes organization, permitted query scope and source/configuration revision. Cap entry size, total memory, expiry and refresh work. Recheck access on retrieval.

Use per-key coalescing, bounded refresh and revision-checked publication. A refresh of revision 8 finishing after revision 9 must not overwrite 9. Comparing only an evictable cache entry is insufficient: eviction can erase that newer watermark. Use immutable revision keys plus an authoritative current-revision lookup, or a non-regressing publication watermark with its own recovery semantics. Commit first, then invalidate/publish; durable notifications may be retried. Define stale eligibility and maximum stale age per read.

Where probabilistic early refresh is useful, with recomputation time `delta > 0`, `beta > 0`, uniform `U in (0,1]`, refresh when `now - delta * beta * ln(U) >= expiry`. It reduces synchronized misses; it does not guarantee one refresher. [Original paper](https://cseweb.ucsd.edu/~avattani/papers/cache_stampede.pdf).

A dependent read states a minimum committed revision. Serve only a sufficiently current view; otherwise read the authority or wait within a deadline. A five-second primary affinity window is a heuristic. Replica checks require replay position on the same history/timeline and the actual read transaction, not merely receipt of WAL. The initial ERP can use primary reads without a replica/cache.

## Migration and acceptance

Expand compatible schema, deploy tolerant readers/writers, backfill restartably, verify totals, cut over, then contract after supported old clients and queued commands are safe. Specify lock budgets, cancellation and failed-index cleanup; online DDL is not lock-free. Do not run a concurrent PostgreSQL index build inside a normal migration transaction. [CREATE INDEX](https://www.postgresql.org/docs/18/sql-createindex.html), [ALTER TABLE](https://www.postgresql.org/docs/18/sql-altertable.html).

Acceptance must include: pooled A→B→missing scope after exception/rollback; cross-tenant foreign reference; competing refunds with distinct keys; lost commit reply; stale cache refill; index/DDL contention on populated data; and old-client queued-command replay. Record unexecuted cases as planned, not passed.
