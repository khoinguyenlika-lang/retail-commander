# System design diagrams

These Mermaid diagrams represent the **proposed architecture**, not an implemented deployment.

## Component architecture

```mermaid
flowchart TB
    subgraph Sources
        INV[Supplier invoices]
        SCN[Offline receiving scanner]
        POS[POS sales]
        CNT[Physical counts]
    end
    subgraph Backend
        API[Ingestion adapters + validation]
        DEDUP[Idempotency and conflict quarantine]
        LEDGER[(Append-only event ledger)]
        PROJ[Inventory projection]
        REC[Reconciliation engine]
        CASE[Discrepancy case service]
    end
    subgraph Review
        UI[Manager review UI]
        AUTH[Approval and audit log]
        POST[Controlled adjustment posting]
    end
    INV --> API
    SCN --> API
    POS --> API
    CNT --> API
    API --> DEDUP --> LEDGER --> PROJ --> REC --> CASE --> UI
    UI --> AUTH
    AUTH -->|Approved only| POST
    POST --> LEDGER
```

## Interrupted scanner replay

```mermaid
sequenceDiagram
    participant S as Offline scanner
    participant A as Ingestion API
    participant L as Event ledger
    participant R as Reconciliation
    S->>S: Persist scan ID and sequence locally
    Note over S,A: Network interruption
    S->>S: Queue additional scans
    Note over S,A: Connectivity restored
    S->>A: Replay unacknowledged events
    A->>L: Insert if source + event ID unseen
    L-->>A: Accepted or already accepted
    A-->>S: Acknowledge durable event IDs
    L-->>R: Trigger updated projection
    R->>R: Keep case uncertain if sequence gaps remain
```

## Manager approval

```mermaid
stateDiagram-v2
    [*] --> Open
    Open --> WaitingForEvents: incomplete receipt
    Open --> NeedsRecount: missing physical count
    WaitingForEvents --> ReadyForReview: evidence complete
    NeedsRecount --> ReadyForReview: verified count available
    ReadyForReview --> Approved: manager approves
    ReadyForReview --> Rejected: manager rejects
    ReadyForReview --> ClosedNoAdjustment: reconciled
    Approved --> PostingPending: idempotent command
    PostingPending --> Posted: downstream confirmation
    PostingPending --> PostingFailed: downstream error
```
