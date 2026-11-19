# Proposed data contracts (v0)

These are design proposals and may change during implementation.

## Canonical event envelope

```json
{
  "event_id": "scan-0180",
  "source_system": "receiving_scanner",
  "event_type": "RECEIPT_SCAN",
  "sku": "SKU-001",
  "location_id": "STORE-01",
  "quantity": 1,
  "unit": "each",
  "device_timestamp": "2026-11-19T10:00:00Z",
  "server_received_at": "2026-11-19T10:03:00Z",
  "source_sequence_number": 180,
  "receiving_session_id": "RCV-001",
  "correlation_id": "PO-001",
  "schema_version": 1
}
```

`event_id` is stable across retries. Uniqueness is `(source_system, event_id)`. A same-key event with a different canonical payload is a conflict. `server_received_at` is assigned by the server, not trusted from clients. Quantities are signed only in the normalized movement projection; raw event types retain domain-specific semantics.

## Reconciliation result

```json
{
  "case_id": "CASE-001",
  "sku": "SKU-001",
  "location_id": "STORE-01",
  "status": "INSUFFICIENT_EVIDENCE",
  "reported_variance": -35,
  "verified_variance": null,
  "uncertainties": [
    "RECEIVING_SESSION_INCOMPLETE",
    "OPENING_BALANCE_UNKNOWN",
    "PHYSICAL_COUNT_NOT_VERIFIED",
    "POS_LOT_ALLOCATION_UNKNOWN"
  ],
  "recommended_actions": [
    "Recover or verify missing scanner events",
    "Verify opening balance and stock count basis",
    "Request independent recount"
  ],
  "adjustment_authorized": false
}
```

`reported_variance` is an externally reported claim, not a verified physical shortage. `verified_variance` remains null until the inputs are compatible and verified.

## Audit record

Record `case_id`, reviewer identity, decision, timestamp, reason, evidence version, and any downstream adjustment command ID. The review and posting are separate operations.
