# Acceptance test scenarios

These are **planned tests**, not executed test results.

| ID | Scenario | Input / fault | Expected behavior |
| --- | --- | --- | --- |
| T01 | Duplicate scanner replay | Same source and event ID delivered twice with identical payload | Exactly one accepted inventory movement; second delivery acknowledged as duplicate |
| T02 | Conflicting duplicate | Same source and event ID, different quantity | Quarantine conflict; do not overwrite original |
| T03 | Offline sequence gap | Scanner sends sequences 1-180 and later 182 | Flag missing 181; receiving session remains incomplete until verified |
| T04 | Out-of-order delivery | Events 3, 1, 2 arrive in that order | Derived totals equal those from 1, 2, 3 when causally independent |
| T05 | Skewed device clock | Scan timestamp is ahead of server time | Retain both timestamps; do not infer cross-system causality from device clock |
| T06 | Delayed POS sale | Sale arrives after initial case generation | Recompute versioned projection and reopen or update case with audit trail |
| T07 | Missing opening balance | Receipt 180, sales 15, no opening snapshot | No definitive expected on-hand or theft conclusion |
| T08 | Invoice mismatch | Invoice 200, scans 180, session interrupted | Mark 20 invoiced units unverified, not stolen |
| T09 | Ambiguous 35-unit report | Invoice 200, scans 180, sales 15, claimed shortage 35 | Flag mixed baselines; request count and opening balance; no verified shortage |
| T10 | POS stock attribution | Sale of 15 with unknown lot allocation | Do not assume sales came from new pallet |
| T11 | Recount resolves case | Verified count matches compatible projected on-hand | Close without adjustment |
| T12 | Approval rejected | Manager rejects proposed adjustment | No posting command emitted |
| T13 | Posting retry | Approved command delivered twice | One downstream adjustment using stable command ID |
| T14 | Late evidence after approval | New receipt event changes case evidence | Preserve original decision; flag for re-review before any new action |
| T15 | Different unit of measure | Cases and eaches mixed without conversion | Reject reconciliation as incompatible until conversion is verified |

## Synthetic fixture: invoice / scanner / POS ambiguity

**Given:** supplier invoice = 200; confirmed scans = 180; POS sales = 15; reported shortage = 35; opening stock = unknown; physical count = unverified; receipt session = interrupted.

**When:** reconciliation runs.

**Then:**

- `verified_variance` is `null`.
- `status` is `INSUFFICIENT_EVIDENCE`.
- `reported_variance` may retain `-35` as an unverified external claim.
- No theft allegation or stock adjustment is generated.
- The case asks for receiving recovery, compatible opening stock, and a verified count.

## Determinism property

For any permutation of the same set of valid, independent event IDs, normalized movement totals must match. Ordering-sensitive operations require explicit causal links and must surface uncertainty if those links are missing.

## Implementation test levels (planned)

- Unit: event validation, idempotency, quantity normalization, status transitions.
- Integration: offline replay, ledger projection, approval-to-posting command.
- Property-based: permutations, duplicates, delayed events.
- End-to-end: synthetic scenario to manager decision with immutable audit record.
