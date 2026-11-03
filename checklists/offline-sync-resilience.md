# Offline-First & Distributed Sync Resilience Checklist

Use this checklist when building or reviewing edge nodes, local-first desktop apps (Tauri/SQLite), and cloud sync gateways.

---

## 1. Local Edge Node Durability (SQLite / Desktop POS)
- [ ] **Transaction Atomicity:** Does every local business mutation (order creation, payment, receipt issuance) commit atomically with its pending sync queue item in one SQLite transaction?
- [ ] **Encryption at Rest:** Is the local SQLite database encrypted with SQLCipher using a key stored in OS-backed secure storage (e.g., Windows Credential Manager / Apple Keychain)?
- [ ] **Authority Lease Validity:** Does the client verify a cryptographically signed offline authority lease before allowing local unlocks or shift actions?
- [ ] **Clock Drift Protection:** Does the local client detect and block material backward clock manipulation? Are monotonic sequence numbers used instead of local wall-clocks for operation ordering?

---

## 2. Sync Protocol & Gateway
- [ ] **Compound Idempotency Constraint:** Does the cloud database enforce a compound unique index on `(organization_id, device_id, operation_id)`?
- [ ] **In-Order Batch Processing:** Does the server apply client mutations in strict `local_sequence` order per device?
- [ ] **Cursor-Based Pull:** Does the client pull cloud changes using an opaque monotonic cursor, committing the batch to SQLite before acknowledging the cursor?
- [ ] **Replay Protection:** Does replaying an existing sync payload return the exact original HTTP status and response payload without side effects?

---

## 3. Conflict & Failure Management
- [ ] **Non-Blocking Rejections:** If an operation in a batch fails validation, are independent operations allowed to continue while only dependent operations are blocked?
- [ ] **Audited Operator Exceptions:** Are rejected transactions surfaced to a visible manager review queue with explicit remediation commands (`void_local_sale`, `reprice_and_resubmit`)?
- [ ] **Exponential Backoff with Jitter:** Does the sync client use randomized, capped exponential backoff (`min(max_backoff, 2^n + jitter)`) to prevent thundering-herd reconnect storms?
- [ ] **Fiscal Window Escalation:** If offline duration approaches the regulatory fiscal window (e.g., ETA e-receipt submission deadline), does the POS display escalating visual warnings and trigger late-submission workflows? Verify the current deadline per jurisdiction/document type and make it configurable.

---

## 4. Schema, Deletion & Observability
- [ ] **Schema Versioning:** Is the local SQLite schema versioned with explicit forward/backward migration rules?
- [ ] **Tombstones:** Are deletes represented as durable tombstones (or an equivalent deletion log) so they sync reliably?
- [ ] **Sync Telemetry:** Are sync lag, retry counts, conflict rates, and DLQ depth observable per tenant and device?
- [ ] **Lease Lifecycle:** Are authority leases time-bound, revocable, and auditable, with defined behavior on expiry?
- [ ] **Upgrade Path:** Can an old client safely sync after a server-side schema or protocol change, or is it forced to upgrade?
