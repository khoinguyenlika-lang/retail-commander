# Architecture and design decisions

## Principles

1. **Evidence before inference.** Separate confirmed events, pending events, and hypotheses.
2. **Source clocks are not authoritative.** Preserve device timestamps and server receipt timestamps; use source sequences and explicit causal references where available.
3. **At-least-once ingestion with idempotent processing.** Offline devices may replay events.
4. **Ledger before projection.** Store normalized source events append-only; build derived inventory views from them.
5. **Human approval for consequential changes.** No autonomous inventory adjustment or theft allegation.

## Proposed components

- **Adapters:** ingest supplier invoices, receiving scans, POS sales, physical counts, and adjustment decisions.
- **Validation and deduplication:** validate schema and unique `(source_system, event_id)` identity; quarantine conflicting duplicates.
- **Event ledger:** append-only records with source metadata, ingestion time, correlation IDs, and immutable payload references.
- **Projection engine:** compute expected on-hand using verified opening stock + verified receipts + transfers in - sales - transfers out +/- approved adjustments.
- **Reconciliation engine:** compare quantities only when SKU, location, unit, time window, and baseline align.
- **Case service:** store evidence links, uncertainty flags, suggested next actions, and review status.
- **Approval service:** record manager identity, decision, reason, and adjustment authorization; downstream posting must be separately confirmed.

## Critical invariants

- An invoice is not proof of physically received stock.
- A scanned receipt is not necessarily complete while the receiving session is interrupted.
- A POS sale does not prove the sold unit originated from a particular pallet or lot unless allocation evidence exists.
- A shortage cannot be inferred without a compatible opening balance, movement window, and independently verified physical count.
- An event replay must not alter totals after its first accepted posting.
- A conflicting duplicate is quarantined rather than silently overwritten.
- A rejected or pending approval cannot post an adjustment.

## Offline recovery

A scanner assigns durable event IDs and monotonically increasing per-device sequence numbers before transmission. On reconnection it retries unacknowledged events. The server accepts new IDs once, acknowledges already accepted identical events, and quarantines same-ID/different-payload conflicts. Sequence gaps are recorded as incomplete evidence; a sequence number alone does not establish ordering across devices.

## Ordering and reconciliation

Maintain both event time and ingestion time. Use source ordering only within its supported scope; link causally related events through references such as `receiving_session_id` or `sale_id`. Reconciliation is scoped to a defined snapshot and watermark. Late events trigger a new version of the projection and case rather than silently rewriting an already reviewed decision.

## Inventory formula

For a consistent SKU/location/unit/window:

`expected_on_hand = verified_opening + verified_receipts + verified_transfers_in - verified_sales - verified_transfers_out + approved_adjustments`

`variance = verified_physical_count - expected_on_hand`

If any required operand is unknown or incompatible, report `INSUFFICIENT_EVIDENCE` instead of calculating a definitive physical shortage.

## Case state machine

`OPEN -> WAITING_FOR_EVENTS | NEEDS_RECOUNT | READY_FOR_REVIEW -> APPROVED | REJECTED | CLOSED_NO_ADJUSTMENT`

A case can be reopened on material late evidence; any previous decision must remain in the audit history. Approval authorizes a proposed adjustment; posting requires a separate idempotent downstream command and confirmation.

## Security and privacy (planned)

Role-based access, least-privilege adapters, audit logs, no secrets in repository, synthetic data for tests, and data-retention controls before production use.

## Non-goals for MVP

Automatic accusations, supplier chargebacks, live ERP writes, machine-learning anomaly scoring, and production-grade multi-tenant compliance.
