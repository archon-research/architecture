# Synome — State

What categories of operational state the Synome stores. This is the schema, not the data itself.

The Synome has two partitions: **[Rules](synome-rules.md)** and **State** (this doc). Rules holds governance-set configuration; State holds operational data. Together they form the Synome — the canonical data layer that Core operates on.

For how the Synome relates to the rest of the architecture, see the [architecture diagram](../diagrams/laniakea-architecture.excalidraw).

---

## What State is

The fast-changing partition of the Synome. Positions, prices, computed metrics, lifecycle records, and audit trail. Holds raw facts ingested from blockchains and derived state produced by Core services applying Rules to those facts. Answers **"what is happening and what happened?"**

---

## State categories

| Category | What it contains | Source |
|---|---|---|
| Positions | Current holdings per agent, per book | Chain events via data ingest |
| Oracle prices | Asset prices, exchange rates | Chain oracles via data ingest |
| Block metadata | Block numbers, timestamps, tx hashes | Chain via data ingest |
| NFAT records | Individual NFAT state (which state, notional, dates, counterparty) | Core services (lifecycle transitions) |
| Attestations | Who attested what, when | Core services (attest workflow) |
| Risk scores | Computed CRR, ER, unit risk weights per agent | Core services (risk calculation) |
| Settlement records | Per-Prime settlement outputs, global aggregation results | Core services (settlement) |
| Breach records | Covenant breaches: when, duration, severity | Core services (risk monitoring) |
| Penalty calculations | Computed penalties from breaches | Core services (settlement) |
| Audit trail | Log of all actions: who did what, when, with what result | All services |

---

## How each category maps to the system

### Positions
Current state of what each agent holds where. Sourced from chain events — data ingest watches blockchains and writes position changes. Core services read positions to compute risk.

### Oracle prices
Asset prices from on-chain oracles. High-frequency, used by risk calculation and settlement. Sourced from data ingest.

### Block metadata
Block numbers, timestamps, transaction hashes. Reference data that links everything back to the chain. Sourced from data ingest.

### NFAT records
The current state of each individual NFAT: what lifecycle state it's in, its notional, term, counterparty, deployment date. The lifecycle *rules* (valid states, valid transitions) live in the Rules partition. The *records* of where each NFAT actually is live here.

### Attestations
Records of attestation events: beacon X attested Y at time T. The *rules* for what requires attestation and who can attest live in the Rules partition. The attestation *records* live here.

### Risk scores
Computed outputs from applying Rules risk formulas to current positions and prices. ER per Prime, CRR per position, unit risk weights. Recomputed frequently. These are derived state, not rules.

### Settlement records
Outputs of each settlement cycle: per-Prime net owed, global aggregation, distribution amounts, fee calculations. Produced by Core services at each settlement boundary.

### Breach records
When a covenant was breached, by whom, for how long, at what severity. Produced by Core services monitoring ER against Rules covenant thresholds.

### Penalty calculations
Penalties computed from breaches using Rules penalty formulas applied to breach records. Feeds into settlement.

### Audit trail
Append-only log of all system actions. Who submitted what, what was accepted/rejected, what was computed. Used for compliance, debugging, and dispute resolution.
