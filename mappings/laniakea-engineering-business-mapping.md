# Laniakea Architecture — Engineering ↔ Business Terminology Mapping

This document maps engineering terms used in the Laniakea architecture to their corresponding Synome framework equivalents. Both audiences reference the same [architecture diagram](../diagrams/laniakea-architecture.excalidraw) and use this mapping as a translation layer.

**Context:** Laniakea is delivered in phases (Foundation 0–4, Factories 5–8, Automation 9–10). The components mapped here exist today and evolve as Laniakea progresses; the mapping itself is durable — terms deepen as later cognitive layers (factories, sentinels, full crystallization) come online, but the underlying engineering ↔ Synome correspondence holds.

## Component Mapping Table

| Component | Function | Engineering Term | Synome / Business Term |
|---|---|---|---|
| **Blockchain** | External source of truth — smart contracts, events, oracles | Public chain state | On-chain protocol state |
| **Data Ingest** | Watches blockchains, scrapes blocks, transactions, and events | Block watcher / indexer (STL) | Blockchain Observer |
| **Synome** | The canonical data layer containing both Rules and State | Source-of-truth platform | Synome (the institutional memory) |
| **Synome — Rules** | Governance-set configuration: formulas, parameters, definitions | Config store / rules engine | Atlas (encoded as specs) |
| **Synome — State** | Operational data: positions, records, events | Operational database (TimescaleDB) | Crystallization Interface / Evidence |
| **Core** | Compute layer: applies Rules to State, produces derived state | Domain services / business logic | Beacon Types (LPLA/LPHA/HPHA) |
| **Access & Routing** | Authenticates requests, enforces permissions, routes to Core or State | API gateway + auth (RBAC) | Relay Beacon / Governance enforcement |
| **Applications** | External apps, dashboards, and integrations built by teams | Client apps / UIs / API consumers | User Interfaces |

## Component Descriptions

### Blockchain — On-chain Protocol State
**What it is:** The external blockchains where smart contracts, price oracles, and protocol transactions live. Not part of Laniakea — Laniakea reads from it.

- **Engineering perspective:** EVM chains (Ethereum, Base, Arbitrum, etc.) hosting PAU contracts, token contracts, oracle feeds
- **Business perspective:** The on-chain state of the Sky protocol — where USDS, BEAMs, and agent contracts live

### Data Ingest — Blockchain Observer
**What it does:** Subscribes to blockchain providers, fetches new blocks, detects chain reorgs, scrapes smart contract events and oracle prices, writes raw data into Synome State.

- **Engineering perspective:** The STL Go service — watcher (WebSocket subscription), backfill workers, oracle price indexers, position trackers. Supports Chainlink, Chronicle, Redstone, and protocol-native oracles.
- **Business perspective:** The system's eyes on the blockchain — observes what's happening on-chain and records it.

### Synome — The Canonical Data Layer
**What it is:** The single source of truth containing both governance rules and operational state. Core reads from it and writes derived state back to it.

The Synome has two partitions:

#### Rules (slow-changing)
**What it holds:** Risk formulas, settlement logic, agent definitions, authority chain, lifecycle rules, token rules, matching rules, phase carve-outs. Changes at governance speed.

- **Engineering perspective:** Config store for governance-set parameters. See [synome-rules.md](../reference/synome-rules.md) for the full schema.
- **Business perspective:** The Atlas encoded in machine-readable form — what the protocol's rules ARE.

#### State (fast-changing)
**What it holds:** Positions, oracle prices, block metadata, NFAT records, attestations, risk scores, settlement records, breach records, penalty calculations, audit trail.

- **Engineering perspective:** Operational database (PostgreSQL/TimescaleDB). Append-only writes, block versioning for reorg handling. See [synome-state.md](../reference/synome-state.md) for the full schema.
- **Business perspective:** The evidence side — what HAS happened and what was computed. Crystallization reads from here and promotes findings into Rules.

### Core — Compute Layer
**What it does:** Reads Rules and State from the Synome, applies business logic, writes derived state back. Handles risk calculation, settlement, NFAT lifecycle, attestation workflows, breach monitoring, auctions, and distributions.

- **Engineering perspective:** Domain services implementing business logic. See [core-services.md](../reference/core-services.md) for the full service catalog.
- **Business perspective:** The regulated Beacons that operationalize Synome constraints:
  - **LPLA Beacon** (Low Power, Low Authority): Read-only verification, no state mutations
  - **LPHA Beacon** (Low Power, High Authority): Deterministic execution of bounded operations
  - **HPHA Sentinel** (High Power, High Authority): Monitors system state, triggers escalations (Phase 9–10)

### Access & Routing — API Gateway + Auth
**What it does:** All requests from Applications pass through here. Authenticates callers, enforces RBAC permissions, then routes: read requests go directly to State, write/action requests go to Core.

- **Engineering perspective:** API gateway with auth middleware, RBAC enforcement, request routing
- **Business perspective:** The governance enforcement point — ensures only authorized actions reach the system. Maps to the Relay Beacon (LPHA, pBEAM) pattern: validates against authority envelope before executing.

### Applications — External Interfaces
**What it is:** The apps, dashboards, and integrations that teams build to interact with the system. Not part of the core architecture — they consume it.

- **Engineering perspective:** Client apps, web UIs, API consumers, third-party integrations
- **Business perspective:** How users (internal teams, institutional investors, governance participants) interact with the protocol

---

## Data Flow Summary

```
Blockchain
    ↑ listens
Data Ingest
    ↓ writes
Synome (Rules + State)
    ↑↓ reads/writes
Core ←── reads Rules
    ↑ writes
Access & Routing
    ↑
Applications
```

---

## Why This Mapping Matters

**For Engineering Teams:**
- Understand how the architecture's components map to Synome concepts for cross-team communication
- Reference concrete services/technologies when discussing abstract Synome roles
- Know which services implement which Synome principles and how the mapping deepens as later phases land

**For Business/Governance Teams:**
- See how the architecture operationalizes Atlas principles through regulated beacons
- Understand that the Synome holds both the rules AND the evidence — Rules is what must be true, State is what has happened
- Discuss policy changes knowing which services implement which constraints

**For Presentations & Synchronization:**
- Use the Synome terminology when discussing governance and long-term architecture
- Use the Laniakea Phase framing when discussing current rollout timelines
- Reference this mapping to translate between team vocabularies
