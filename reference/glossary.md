# Glossary

Definitions of terms used across this repository. Cross-references both the **business vocabulary** (Sky Ecosystem, Synome, Laniakea — sourced from `laniakea-docs`) and the **engineering vocabulary** used in implementation repos (`stl`, `synome`, `infrastructure`).

When a term has both a business and an engineering form, both are listed and linked.

---

## Top-level Concepts

| Term | Definition |
|---|---|
| **Sky Ecosystem** | Decentralized stablecoin ecosystem powering USDS (~$9.9B supply, evolved from MakerDAO). The business that engineering serves. |
| **USDS** | Sky's primary stablecoin (1:1 USD peg, overcollateralized). Successor to DAI. |
| **sUSDS** | ERC-4626 savings token. Yield accrues via increasing exchange rate. |
| **SKY** | Governance token of the protocol. Fixed supply, deflationary via buybacks. |
| **Laniakea** | The current multi-year **infrastructure upgrade** of Sky toward automated capital deployment at scale. Delivered in 11 phases (0–10). |
| **Synome** | Sky's **source-of-truth** layer: the data and governance system that holds the Atlas, agent directives, and crystallized rules. Implemented in the `synome` engineering repo. |
| **Atlas** | Sky's **governance constitution** — the set of principles, axioms, and formulas that define how the protocol must operate. Encoded as Python specs in `next-gen-atlas`. |
| **The Hearth** | The teleological framework grounding Sky's long-term mission (three immutable commitments to natural life, the solar system, and natural sovereignty). |

## Synomic Agents (the institutional layer)

A **Synomic Agent** is a durable, ledger-native entity that can own assets and make binding commitments. Agents are organized into ranks:

| Term | Rank | Definition |
|---|---|---|
| **Core Council** | 0 | Sovereign governance body. |
| **Guardian** | 1 | Directly regulated by Core Council; provides accord to Primes/Generators. |
| **Core Controlled Agent** | 1 | Tokenless agent administered directly by Core Council (used for legacy positions). |
| **Recovery Agent** | 1 | Crisis agent activated when a Guardian collapses; temporary. |
| **Prime** | 2 | Capital deployer accordant to a Guardian. Receives capital from Sky Protocol and deploys via Halos. Each has its own treasury and governance token. Examples: Spark, Grove, Keel. |
| **Generator** | 2 | Manages the interface with the stablecoin contract; deploys capital into Primes via ERC-4626 vaults. Issues `srUSDS`. |
| **Halo** | 3 | Investment-product wrapper administered by a Prime. Three Class types: Portfolio (LCTS), Term (NFAT), Trading (AMM). |
| **Folio Agent** | 3 | Single-owner, tokenless supply-side holding structure. Used by Growth Staking participants. |

## Beacon Framework

| Term | Definition |
|---|---|
| **Beacon** | A Synome-registered, enforceable action aperture through which an embodied agent affects the external world. Always: registered, scoped, anchored, observable, revocable. |
| **Teleonome** | A private, goal-directed AI system. "Dark by default" — invisible to the protocol unless it operates beacons. |
| **Authority Envelope** | The scope of permitted actions for a beacon (rate limits, approved targets, attestation obligations). |
| **LPLA** | Low Power, Low Authority — independent beacons (reporting, simple data exchange). Example: `lpla-verify`, `lpla-checker`. |
| **LPHA** | Low Power, High Authority — "Keepers": deterministic rule executors operating Synomic Agents. Examples: `lpha-relay`, `lpha-nfat`, `lpha-attest`, `lpha-council`, `lpha-lcts`, `lpha-rate`, `lpha-auction`. |
| **HPLA** | High Power, Low Authority — sophisticated peer-to-peer beacons trading their own capital. Example: `hpla-trade-{actor}`. |
| **HPHA** | High Power, High Authority — Sentinels and high-authority governance beacons. Continuous real-time control. |
| **Sentinel** | A distinguished HPHA subclass. Four types: `stl-base` (Baseline), `stl-stream` (Stream), `stl-warden` (Warden), `stl-principal` (Principal). |
| **BEAM** | Bounded External Access Module — on-chain authorized role that gives a beacon High Authority. |
| **pBEAM** | Process BEAM. Direct execution authority (held by Relay Beacon / `lpha-relay`). |
| **cBEAM** | Configurator BEAM. Configuration authority — sets rate limits, onboards targets. Held by Relay Beacons; granted by Council Beacon. |
| **aBEAM** | Admin BEAM. Administrative authority — registers PAUs, grants cBEAMs. Held by Council Beacon (HPHA). 14-day timelock on additions. |

## Smart-contract Primitives

| Term | Definition |
|---|---|
| **PAU** | Parallelized Allocation Unit. A Synomic Agent's on-chain control surface: Controller + ALMProxy + RateLimits. |
| **Diamond PAU** | The Phase-1+ PAU pattern using EIP-2535 diamond proxy architecture. Enables modular facets (`NfatDepositFacet`, `CoreHaloFacet`, etc.) without full redeployment. |
| **Configurator Unit** | The aBEAM/cBEAM control stack: BEAMTimeLock → BEAMState → Configurator. Enables spell-less Prime operations. |
| **SORL** | Second-Order Rate Limit. Constraint on how fast rate limits themselves may increase (currently 25% per 18h). |
| **LCTS** | Liquidity Constrained Token Standard. Queue-based system for capacity-constrained token conversions. Powers Portfolio Halos and `srUSDS`. |
| **NFAT** | Non-Fungible Allocation Token. ERC-721 representing an individual bespoke deal in a Term Halo. Each NFAT has its own duration, size, yield within a Halo's "buybox". |
| **Buybox** | The acceptable parameter ranges (duration, yield, size, counterparty class) within which NFAT deals can be auto-executed under a Halo's legal framework. |
| **Halo Class** | Grouping of Halo Units sharing the same PAU, beacon, and legal framework. |
| **Halo Unit** | An individual product within a Halo Class (e.g., a senior or junior tranche). |
| **srUSDS** | Senior Risk Capital token issued by the Generator via LCTS. |
| **TEJRC** | Tokenized External Junior Risk Capital (Prime-level, via LCTS). |
| **TISRC** | Tokenized Isolated Senior Risk Capital (Prime-level, via LCTS). |
| **OSRC** | Originated Senior Risk Capital. Core Council-set rate pre-Phase-9; auction-allocated once `stl-base` is live. |

## Settlement & Risk

| Term | Definition |
|---|---|
| **Daily Settlement Cycle** | The Phase 3+ cadence: Active Window 16:00→13:00 UTC, Processing (Lock) 13:00→16:00 UTC, Moment of Settlement at 16:00. |
| **Tug-of-war** | Algorithm for allocating duration capacity to existing reservations. |
| **Lindy Duration Model** | Liability duration measurement based on USDS lot ages (the longer held, the longer expected to be held). |
| **ALDM** | Asset-Liability Duration Matching. Matches asset SPTP to liability duration buckets. |
| **SPTP** | Stressed Pull-to-Par. Time until an asset converges to its fundamental value under stress. |
| **JRC / SRC** | Junior / Senior Risk Capital tiers. |
| **IJRC / EJRC** | Internal / External Junior Risk Capital. |
| **FLC** | First Loss Capital. The 10% of total JRC posted by the Prime itself. |
| **ORC** | Operational Risk Capital. Independent capital charge sized as `Rate Limit × TTS`. |
| **TTS** | Time to Shutdown. Warden detection latency; bounds maximum damage from a rogue sentinel. |
| **Encumbrance Ratio** | `Required Risk Capital / Total Risk Capital`. Target ≤90%. |
| **Crystallization** | The process by which probabilistic / observed evidence becomes a binding deontic axiom in the Atlas. |

## Engineering — STL (block watcher / data pipeline)

| Term | Definition |
|---|---|
| **STL block-data service** | The Go service that watches blocks and indexes protocol state. *Not the same as the Phase-9 sentinel `stl-base`* — it is the Phase-1 data-ingestion layer that any beacon (`lpla-verify`, `lpha-attest`, etc.) depends on. |
| **Watcher** | The live-data service: subscribes to a managed RPC provider over WebSocket, fetches receipts / traces / blobs, detects reorgs, publishes events to a FIFO message stream. |
| **Backfill** | Service that fills gaps in block data via HTTP polling when the watcher has been down. |
| **Backup Worker** | Archives raw block data to durable object storage for long-term storage and audit. |
| **Position Tracker** | Per-protocol indexer that consumes block events and writes position state to the time-series database. One tracker exists per source protocol. |
| **Oracle Price Worker** | Consumes block events and writes oracle-price snapshots to the time-series database. Uses batched RPC for efficiency. |
| **Block Versioning** | `(block_number, block_version)` pair. `block_version` increments on chain reorgs so downstream consumers can reconstruct state at any point. |
| **Append-only convention** | Database writes use an upsert-no-overwrite pattern (`ON CONFLICT DO NOTHING`). Reorgs are represented as new versioned rows, not deletes. |

## Engineering — Synome (source-of-truth platform)

| Term | Definition |
|---|---|
| **Synome (the platform)** | The Python platform implementing the source of truth — parser, graph-DB integration, Explorer UI, HTTP backend, code-gen. |
| **Atlas spec submodule** | Git submodule containing the actual specs and the spec-validation Python package. |
| **Spec** | A formula or rule encoded as Python in a restricted subset (no classes, lambdas, comprehensions). Validated by a `flake8` plugin and parsed into a symbolic-math representation. |
| **Graph database (Synome)** | A branch-aware (Git-backed) graph database that stores identities, formulas, and dependencies. All identity reads are scoped to a branch. |
| **Explorer UI** | Frontend for browsing identities, formulas, dependencies, branch history, and branch comparison. |
| **Symbolic representation** | Synome stores formulas as symbolic-math expressions, renderable to LaTeX (for the UI) and to executable Python (for runtime). |

## Engineering — Infrastructure (platform)

| Term | Definition |
|---|---|
| **Production / Staging clusters** | Two managed Kubernetes clusters of equivalent shape, in a single primary region. Staging mirrors production for change validation. |
| **Hub-and-spoke GitOps** | A single hub cluster runs the GitOps controller and reconciles desired state into both clusters. Each cluster is otherwise independent. |
| **App-of-Apps** | The GitOps bootstrap pattern: one root application enumerates per-team / per-environment applications, which in turn enumerate workloads. |
| **Node autoscaler** | Provisions ARM64 nodes on demand, prioritising spot capacity with on-demand fallback. |
| **Secret sync** | A controller that mirrors secrets from a managed secret store into the cluster, with workload-scoped IAM. No long-lived credentials in cluster manifests. |
| **Mesh-based private network** | Workstation, contractor, and out-of-band cluster access flow through a WireGuard-based identity-aware mesh. No public-internet exposure for control planes. |
| **Read-only contractor proxy** | A bastion-hosted connection pooler that exposes a read-only role on a database replica to authorised external auditors. Authentication is identity-bound; no shared passwords. |

---

## Cross-references

| If you see this term... | Look here |
|---|---|
| Beacon classification (LPLA/LPHA/HPLA/HPHA) | `reference/synome-concepts-for-engineers.md` |
| BEAM hierarchy and authority flow | `reference/security-governance-model.md` |
| How Phase 1 components map to Synome roles | `mappings/synome-engineering-business-mapping.md` |
| What technology each component uses | `reference/technology-stack-matrix.md` |
| What's built today | `roadmap/phase-1-status.md` |
| What changes in later phases | `roadmap/laniakea-phase-evolution.md` |
