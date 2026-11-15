# Offline-First & Distributed Sync Resilience Checklist

Use this checklist for offline durability and recovery. The [offline POS workflow](../references/offline-pos-workflow.md) gives acceptance scenarios for Universal ERP.

---

## 1. Local Edge Node Durability (SQLite / Desktop POS)
- [ ] **Transaction Atomicity:** Do paid sale, tender, immutable receipt data, print job and sync command commit together before dispatch? Does an injected failure before commit leave no partial database effect and trigger the approved reconciliation path if cash changed hands?
- [ ] **Encryption at Rest:** Is the local SQLite database encrypted with SQLCipher using a key stored in OS-backed secure storage (e.g., Windows Credential Manager / Apple Keychain)?
- [ ] **Authority Lease Validity:** Does the client verify the scope and expiry of the signed lease for each action? Test renewal, expiry, and the project rule for actions performed before the device learned of revocation.
- [ ] **Clock Drift Protection:** Does the local client detect and block material backward clock manipulation? Are monotonic sequence numbers used instead of local wall-clocks for operation ordering?

---

## 2. Sync Protocol & Gateway
- [ ] **Compound Idempotency Constraint:** For Universal ERP, are `(organization_id, device_id, device_epoch, operation_id)`, the canonical versioned fingerprint, outcome, effects, and outbox committed atomically? Does changed content under the same key go to review without creating another effect?
- [ ] **Device Epoch:** Are `local_sequence` and uniqueness scoped to device and epoch? Before starting a new epoch, reconcile pending and previously applied operations by stable business references. Does retention cover the supported outage, backup recovery and late arrivals?
- [ ] **Ordering:** Does the server request recovery of missing sequence entries and distinguish them from recorded business rejections whose dependents need review?
- [ ] **Cursor-Based Pull:** Does the client pull cloud changes using an opaque monotonic cursor, committing the batch to SQLite before acknowledging the cursor?
- [ ] **Replay Protection:** After enforcing caller scope, does an identical retry return the recorded outcome? Test lost response after commit and simultaneous retries using real persistence.

---

## 3. Conflict & Failure Management
- [ ] **Non-Blocking Rejections:** If an operation in a batch fails validation, are independent operations allowed to continue while only dependent operations are blocked?
- [ ] **Audited Operator Exceptions:** Are disputed effects visible to the right operator with reason and stable references? Preserve finalized sales and use linked, authorized refunds or adjustments. Check transfer claims under a shared receipt cycle after unit normalization.
- [ ] **Failure Handling:** Do transient failures use bounded exponential backoff and jitter, invalid inputs take a recorded failure path, and uncertain printer or card results go through verification before any repeat? Test a marked reprint with the original receipt number and unchanged totals.
- [ ] **Fiscal Window Escalation:** If offline duration approaches the regulatory fiscal window (e.g., ETA e-receipt submission deadline), does the POS display escalating visual warnings and trigger late-submission workflows? Verify the current deadline per jurisdiction/document type and make it configurable.

---

## 4. Schema, Deletion & Observability
- [ ] **Schema Versioning:** Is the local SQLite schema versioned with explicit forward/backward migration rules?
- [ ] **Tombstones:** Are deletes represented as durable tombstones (or an equivalent deletion log) so they sync reliably?
- [ ] **Sync Telemetry:** Are sync lag, retry counts, conflict rates, and DLQ depth observable per tenant and device?
- [ ] **Lease Lifecycle:** Are authority leases time-bound, revocable, and auditable, with defined behavior on expiry?
- [ ] **Upgrade and Restore:** Test a real backup with pending commands, an older supported client and upload before an incompatible update. Verify that a rewound print queue cannot automatically print an uncertain original again.
- [ ] **Operator State:** Can the POS show local persistence, pending cloud acknowledgement and print review independently, with actionable reasons and references?
