# Laniakea Architecture — Engineering ↔ Business Terminology Mapping

This document maps engineering terms used in the Laniakea architecture to their corresponding Synome framework equivalents. Both audiences reference the same architecture diagram and use this mapping as a translation layer.

**Context:** Laniakea is delivered in phases (Foundation 0–4, Factories 5–8, Automation 9–10). The components mapped here exist today and evolve as Laniakea progresses; the mapping itself is durable — terms deepen as later cognitive layers (factories, sentinels, full crystallization) come online, but the underlying engineering ↔ Synome correspondence holds.

## Component Mapping Table

| Component | Function | Engineering Term | Synome / Business Term |
|---|---|---|---|
| **Entry Point** | Users authenticate and express intent(e.g., Attest UI) | Client app | User Interface |
| **Request handler** | Validates requests, enforces governance policy | API gateway | Relay Beacon (LPHA, pBEAM) |
| **Access Control** | Authenticates users (SIWE/MetaMask), authorizes access, manages roles | Identity & Access | Governance |
| **Synome** | Holds the Atlas, agent registry, beacon registry, formulas, and governance rules. Read by Core; never written to by Core. | Source-of-truth platform | Synome (the source-of-truth layer) |
| **Core** | Implements business logic | Domain | LPLA Beacon (read-only), LPHA Beacon (executor), HPHA Sentinel (monitoring) |
| **Data** | Records verified state, position data, risk calculations, attestations, audit trail | Data Layer | Data |

## Component Description

### API Gateway ↔ Relay Beacon (LPHA, pBEAM)
**What it does:** Accepts incoming execution requests, validates against an authority envelope (rate limits, approved targets), routes requests through Controllers, enforces governance constraints.

- **Engineering perspective:** API Gateway pattern that implements bounded external access—validates requests, enforces rate limiting, routes to appropriate services
- **Business perspective:** Relay Beacon is the regulated entry aperture (LPHA: Low Power, High Authority) that translates external requests into validated Synome operations. pBEAM (Process BEAM) executes operations within governance-defined bounds. Governed by Council Beacon (aBEAM) which controls what Relay Beacons can do.

### Identity & Access ↔ Governance
**What it does:** Authenticates users (validates credentials), manages roles and permissions, decides what each user/service can access.

- **Engineering perspective:** Auth service with SIWE support, RBAC enforcement
- **Business perspective:** Language Intent includes governance rules about who can take what action—this service enforces those intent rules

### Synome ↔ Source-of-truth Platform
**What it does:** Stores and serves the Atlas (specs, formulas, deontic rules), the agent registry, and the beacon registry. Core services read from Synome to know what the rules are; Core does not write to Synome. Spec changes go through human review.

- **Engineering perspective:** Python platform built on a branch-aware graph database with Git-backed specs. Specs are written in a restricted Python subset (no classes, lambdas, or comprehensions), validated by a `flake8` plugin, and parsed into a symbolic-math representation that renders to LaTeX (for the UI) or to executable Python (for runtime). The Explorer UI surfaces identities, formulas, dependencies, and branch comparisons.
- **Business perspective:** Synome is the institutional memory — the canonical, branch-versioned record of what the protocol's rules ARE. The Atlas lives here. **Crystallization** is the path by which probabilistic evidence promotes into binding Atlas axioms; that promotion writes INTO Synome, not into the operational data store. In Phase 1 the path is fully human-reviewed; later phases automate parts of it.

### Domain ↔ Beacon Types (LPLA/LPHA/HPHA)
**What it does:** Implements core business logic—verifying state, computing positions, attesting to results, monitoring system health.

**Beacon Classification:**
- **LPLA Beacon** (Low-Privilege, Low-Authority): Read-only verification logic, no state mutations. Example: Verify service.
- **LPHA Beacon** (Low-Privilege, High-Authority): Deterministic execution of bounded operations, writes verified results. Example: NFAT (position computation), Attest (attestation signing).
- **HPHA Sentinel** (High-Privilege, High-Authority): Monitors system state, can trigger escalations and emergency actions.

- **Engineering perspective:** Microservices implementing hexagonal architecture (ports + adapters). Services read from State Cache, process domain logic, write to both Event Stream and Data Store.
- **Business perspective:** Regulated Beacons operationalize Synome constraints—they execute only operations permitted by Atlas/Language Intent, enforce Deontic Skeleton rules, and coordinate through Probabilistic Mesh

### Data Store ↔ Crystallization Interface / Artifact Hierarchy
**What it does:** Persistent record of verified state — positions, risk calculations, NFAT records, attestations indexed from chain events, and the engineering audit trail. Append-only; never overwrites history.

- **Engineering perspective:** PostgreSQL (TimescaleDB), S3 archives, immutable append-only writes (`ON CONFLICT DO NOTHING` pattern). Block versioning preserves state across reorgs.
- **Business perspective:** The Crystallization Interface captures the frozen, verified snapshots of system state (artifacts). The Artifact Hierarchy describes how state is layered (config → agent → instance). This is the **evidence side** of the system — what *has happened* — and is distinct from Synome, which holds what the rules *are*. Crystallization itself (the promotion of evidence into binding Atlas axioms) reads from this layer and writes into Synome; soft knowledge becomes hard rules in Synome, not here.

---

## Supporting Components

### Data Ingest ↔ Blockchain Observer
**What it does:** Subscribes to blockchain providers, fetches new blocks, detects chain reorgs, populates ephemeral state for Domain services.

- **Engineering perspective:** Watcher service with WebSocket subscription, reorg detection
- **Business perspective:** Observer perceives blockchain state changes and alerts the system.

### State Cache (Internal)
**What it does:** Ephemeral storage of active block data and computation state. Read by Domain services; written by both Data Ingest and Domain services during processing.

- **Engineering perspective:** Redis, high-speed lookup, temporary working memory, survives pod restarts but not cluster failures
- **Business perspective:** Working memory of the system—temporary, losable state used for fast decision-making during computation. Not part of the formal Crystallization Interface (frozen state).

---

## Why This Mapping Matters

**For Engineering Teams:**
- Understand how the architecture's components map to Synome concepts for cross-team communication
- Reference concrete services/technologies when discussing abstract Synome roles
- Know which services implement which Synome principles and how the mapping deepens as later phases land

**For Business/Governance Teams:**
- See how the architecture operationalizes Atlas principles through regulated beacons and crystallization
- Understand that Synome holds the rules and the Data Store holds the evidence — Crystallization is the path from evidence to binding Atlas axioms, writing into Synome
- Discuss policy changes knowing which services implement which constraints
- Understand the roadmap: Foundation builds the institutional layer; Factories make agent onboarding repeatable; Automation adds Sentinels and self-regulating capability

**For Presentations & Synchronization:**
- Use the Synome terminology when discussing governance and long-term architecture
- Use the Laniakea Phase framing when discussing current rollout timelines
- Reference this mapping to translate between team vocabularies

---
