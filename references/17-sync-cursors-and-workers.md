# Reliable workers and complete synchronization streams

Read when implementing outbox delivery, retries, catalog pull, snapshot bootstrap or reconnect. Recipes below are derived designs requiring implementation validation. Preserve the project's authority, lease, business-identity and compatibility rules. [Evidence](primary-sources.md).

## Worker ownership and delivery

Store event ID, organization, event/schema version, aggregate identity/revision, payload, status, attempt count, next attempt, owner, lease expiry and claim generation. A database outbox is durable; an in-memory channel is only a wake-up optimization. Polling must recover missed notifications.

**Scoped queue access:** select the authorized organization before payload access and bind its database transaction. A scheduler may enumerate minimal authorized routing metadata; it must not gain unrestricted business-table access. Isolate scheduler privileges, worker payload access and migration privileges. Do not solve RLS failures with BYPASSRLS.

Within that scope, claim due pending or expired jobs in a short transaction:

```sql
WITH picked AS (
  SELECT organization_id, event_id
  FROM outbox_events
  WHERE organization_id = :organization_id
    AND next_attempt_at <= clock_timestamp()
    AND (status = 'pending' OR
         (status = 'processing' AND lease_until < clock_timestamp()))
  ORDER BY next_attempt_at, event_id
  FOR UPDATE SKIP LOCKED
  LIMIT :batch_size
)
UPDATE outbox_events o
SET status = 'processing', owner_id = :worker_id,
    claim_generation = o.claim_generation + 1,
    lease_until = clock_timestamp() + :lease_interval,
    attempt_count = o.attempt_count + 1
FROM picked p
WHERE (o.organization_id, o.event_id) = (p.organization_id, p.event_id)
RETURNING o.*;
```

Commit the claim before external I/O. Renew, complete, reschedule and quarantine with conditions on organization, event, owner, generation, processing status and unexpired lease. Zero affected rows means ownership lost. Use DB time for claims. Limit batch size to work the worker can start/renew; define renewal failure and process restart behavior. `SKIP LOCKED` is appropriate to this queue, not a consistent reporting query. [PostgreSQL SELECT](https://www.postgresql.org/docs/18/sql-select.html).

An expired W1 can still complete a request already sent while W2 owns the job. Database claim fencing protects job transitions, not the external effect. For a database consumer, commit a unique `(organization_id, consumer_name, event_id)` inbox record/fingerprint with the consumer's effect and outcome. For a provider, use its stable reference/idempotency support and reconcile uncertainty before any repeat. Never promise universal exactly-once external execution.

Preserve a late provider observation under its stable attempt/reference in an append-only evidence record. Losing a claim must not discard evidence that money moved; recording that observation does not authorize the stale worker to complete or overwrite the successor's job. Reconciliation resolves the job against authoritative provider evidence.

Per-aggregate order is separate from claim order: supply expected revisions and hold/recover a gap, or use a serialized aggregate lane. Independent aggregates may progress. A permanent invalid message needs an audited review outcome; a dependency gap is not automatically permanent. Bound attempts/deadlines and jitter transient retries; redrive preserves event/effect identity and requires the applicable permission. [Retry analysis](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/).

## Commit-complete cursors

An ordinary sequence, timestamp or UUID is not a safe change-stream high-water mark. Counterexample: T1 allocates 101 and pauses; T2 commits 102; a client advances to 102; T1 commits 101 later. A subsequent `id > 102` skips it. [Sequence behavior](https://www.postgresql.org/docs/18/functions-sequence.html); the counterexample is a derived concurrency analysis.

For a bounded catalog/configuration stream, a simple initial design is a pre-existing head row per organization/stream. Every relevant writer locks it before changing published state, increments it transactionally, and stores the delta/tombstone in the same transaction. Hold the head lock until commit. A rolled-back publication exposes neither its state nor a new head. Different organizations proceed independently. Do not put sales ingestion behind this configuration lock; measure hot-stream contention before expanding the scheme.

Alternative: a properly managed CDC stream and snapshot boundary. It adds slot lifecycle, WAL-retention, duplicate handling and restore/failover work; do not adopt it merely to avoid specifying a cursor. [Logical decoding](https://www.postgresql.org/docs/18/logicaldecoding-explanation.html).

## Snapshot-to-delta bootstrap

1. Authenticate device, organization and allowed catalog scope. Materialize the entire scoped state and its head `H` from one consistent database snapshot. Commit materialization before serving pages; do not hold a database transaction while a phone downloads them.
2. Serve immutable pages bound to snapshot ID, organization/scope generation, schema version, `H`, expiry and continuation. Tokens are opaque and validated server-side; token possession grants no authority. Provide manifest counts/hashes for completeness.
3. Stage pages into a separate encrypted local catalog generation. A crash keeps the previous active generation. Never replace the entire POS database or delete pending financial/print/sync facts to install a catalog.
4. On complete validation, atomically activate the generation and cursor `H`. Pull deltas after `H`; apply each batch, tombstones and next cursor in the same local transaction. Replaying a batch is safe.
5. Freeze an upper head per catch-up pass so pagination has a bounded endpoint. Renew work with a later head after completing that pass. Updates signal availability; durable pull recovers missed signals.
6. If history has expired, return an explicit resnapshot outcome. Retain/repin history for an in-flight snapshot or expire it before the needed delta range disappears. Permission/scope changes invalidate affected tokens and require safe cache retirement, without deleting device-owned facts.

Every publication that affects offline behavior must participate in the chosen stream or have its own documented boundary. Tombstone/history cleanup is based on supported outage, update compatibility and active snapshot lifetimes; 72 hours of sales authorization is not proof that 72 hours is a sufficient recovery horizon.

## Reconnect and operator truth

Upload committed operations with their original rule revisions, keys and fingerprints. A new catalog cannot silently reprice a paid offline sale. Check actual effective-time and permission policy; recorded rejections/review are durable outcomes, not pretend successful effects. Preserve per-device gap/dependency recovery and cross-epoch business reconciliation.

Bound bytes, operation count, per-device and per-tenant concurrency, global DB work and report work. Admit according to trusted tenant/device identity, with a bounded anonymous path; per-process rate limits multiply when API instances scale. Avoid unbounded limiter partitions from user input. Report overload truthfully: never acknowledge persistence before durable storage or report unapplied work as applied. Durable receipt, dependency wait, review and recorded rejection remain distinct outcomes. Fair scheduling must let a healthy tenant progress beside a large reconnecting tenant. [ASP.NET Core admission controls](https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit?view=aspnetcore-10.0).

Expose local persistence, central command outcome, posted effect, print outcome, review/dependency status and backup age independently. A durable review or dependency-wait outcome can retire the transport retry only when the contract assigns durable retry/recovery responsibility to the server; retain original identity/evidence and unresolved business status. It does not establish a posted effect or close business reconciliation. Measure transport progress and business reconciliation separately.

## Required failure scenarios

- W1 pauses past lease, W2 claims, W1 resumes: no stale completion/reschedule; duplicate delivery produces one consumer effect.
- Consumer commits but response/worker acknowledgement disappears: replay recovers the outcome.
- Snapshot races with updates/deletes; T1 commits after T2; scope changes mid-download: no skipped authorized changes or scope leak.
- Crash during page staging, activation, delta commit or acknowledgement: old generation remains usable or the committed new one resumes correctly.
- Cursor/snapshot expires while unsent sales exist: resnapshot preserves those sales and their identities.
- One tenant reconnects noisily alongside ordinary sales; missing dependencies and poison records stay visible without blocking independent work.
