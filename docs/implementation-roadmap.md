# Implementation roadmap and Devin evaluation

## Milestone 0 — Public specification (current)

Publish problem definition, architecture, contracts, Mermaid diagrams, and test scenarios. **No working implementation is claimed.**

## Milestone 1 — Event core (planned)

Build event API, validation, durable storage, duplicate handling, and synthetic fixture loader. Demonstrate T01-T05.

## Milestone 2 — Reconciliation (planned)

Implement projection, baseline checks, uncertainty flags, and versioned discrepancy cases. Demonstrate T06-T11 and T15.

## Milestone 3 — Human review (planned)

Build a simple review UI, authorization audit, and idempotent adjustment-command simulation. Demonstrate T12-T14.

## Milestone 4 — Evaluation (planned)

Record which tasks were delegated to Devin, human review and rework needed, tests passing, elapsed time, and known failures. Publish results only after measuring them.

## Suggested initial Devin task

> Implement a minimal service that ingests the canonical event envelope, enforces idempotency by `(source_system, event_id)`, quarantines conflicting duplicates, and exposes deterministic projections. Use only synthetic data. Add tests T01-T05. Do not implement automatic inventory adjustments.

## Definition of done for MVP

- Synthetic end-to-end scenario runs locally.
- All agreed acceptance tests pass with evidence.
- Discrepancy case exposes source event references and uncertainty.
- Manager decision is audited; no adjustment occurs without approval.
- README reflects actual implemented features, limitations, and reproducible run instructions.
