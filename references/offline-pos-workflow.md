# Offline POS workflow

Derived application of reliable messaging and transaction boundaries. The examples use Universal ERP's approved design. For another project, identify its authority, capabilities and recovery policy first.

## Transaction and identity boundaries

- Save the cash sale, tender, immutable receipt data, print job and sync command together in SQLite. A failure before commit leaves none of those effects. After commit, payment is a recorded fact and print and upload can proceed separately. Workflows that accept cash before a failed commit need an operator reconciliation path.
- A retry uses `(organization_id, device_id, device_epoch, operation_id)`. Validate actor, organization, branch and device scope. Fingerprint a versioned canonical representation of the meaningful command fields, including scope and versions when part of the command. Exclude transport details such as trace IDs and retry times. Define exact money, unit and encoding rules in the contract and check shared hash vectors.
- Atomically decide retry ownership under a database uniqueness constraint. Persist the outcome, newly posted effects, and associated outbox event in the same server transaction. A matching retry returns the recorded outcome. The same key with a different fingerprint opens review without another posting. Concurrent retries and a lost response after commit produce one sale and payment.
- Reconcile a restored copy or a manual recovery by stable sale ID or receipt reference within its scope. Give each tender or refund its own stable effect identity so multiple legitimate payments on one sale remain distinct. Keep the original identity and outcome of known operations and preserve unmatched evidence for review. Reconcile the prior epoch and pending work before issuing new operations in the new epoch. A reset sequence or expired reply cache cannot authorize a second posting of the same effect.
- Order commands within a device and epoch. Ask for missing sequence entries. A command that was received and has a recorded business rejection is different from a missing command. Its dependents wait or go to review, while later independent commands can proceed according to policy. Apply each pull batch and its cursor in one transaction.

## Transfer claims and settled facts

- Retry keys identify transport attempts. For transfer receipt, match the underlying claim by organization, transfer, and server-issued receipt cycle. Check the shipment revision and compare quantities after normalization. Preserve two equivalent claims in the audit trail and post their stock effect once. Differing claims go to review for the same cycle, even after an earlier posting. Apply the approved hold and any linked adjustment under the transfer transaction, keeping movements that already occurred.
- Verify duplicate `8` and conflicting `8` versus `10` confirmations across devices using different operation IDs. With `3` units already released and `8` ultimately approved, add only `5`. Test concurrent approvals and retries against the real database.
- For a sale, refund or payout performed under a valid offline lease before the device learned of revocation, keep the actual effect once and open review under the project rule. A transfer confirmation under the same conditions preserves evidence and holds approval of the quantity. An expired lease may permit a draft only where the action policy says so.
- Keep a paid sale, issued receipt and closed shift intact. Record authorized corrections or refunds with stable references to the originals. Validate the remaining refundable amount in the refund transaction. Follow the responsible-device rule for cash refunds and the policy for eligible refunds that occur in a later open shift.

## Failure drills

| Injected failure | Evidence to check |
| --- | --- |
| Process exits before local commit | Sale, tender, receipt, print job and queued command all roll back. If cash physically changed hands, exercise the reconciliation procedure. |
| Server commits and response is lost | The local sale remains saved and sync pending. Retrying returns the recorded result with one sale, tender effect and associated outbox event, even though the order is already paid and its current state would reject a new payment. |
| Backup restored or old POS upgraded with pending work | Inspect pending identities and references, match previously applied work, and recover the remainder without loss. Verify supported old-client contracts and the queue upload path before incompatible upgrade. |
| Printer times out after dispatch or a backup rewinds job state | Treat the outcome as uncertain unless authoritative spooler or printer evidence resolves it. Preserve the attempt. An authorized marked reprint uses the same receipt number and leaves sale and payment counts intact. |
| Card terminal outcome is uncertain | Verify the original payment reference and terminal records before allowing a new charge. Preserve confirmed and unresolved outcomes separately. |
| Several clients reconnect together | Stagger transient retries and measure oldest pending age, checkout latency and queue recovery. Invalid input and recorded business rejections take their review path without repeatedly retrying. |

Expose local persistence, cloud acknowledgement and print outcome independently. A sale can be saved locally, queued for upload, and waiting for printer review simultaneously. Show the reason, stable reference and available action for each pending item.
