# Architecture Trade-offs

The major decisions made for Laniakea Phase 1, the alternatives that were considered, and the principles that drove the final choice. Conceptual; specific products, accounts, regions, and instance types are not named.

Each entry is structured the same way:

> **Decision** — the choice made.
> **Considered** — the realistic alternatives.
> **Why** — the principle or constraint that decided it.
> **Cost** — what we give up.
> **When to revisit** — the signal that would change the answer.

---

## 1. Append-only data with block versioning, not in-place reorg correction

**Decision.** Every block-derived row is keyed by `(block_number, block_version)`. Reorgs increment `block_version` and write new rows. Nothing is deleted or mutated.

**Considered.**
- *In-place correction*: when a reorg is detected, update or delete the affected rows.
- *Soft-delete with `valid_from / valid_to`*: keep history but require every read to filter.

**Why.** Audit reconstruction must be exact. Replay safety must be unconditional. The append-only convention buys: idempotent indexer recovery, safe duplicate writes from live + backfill, lossless reorg handling, and a simple reasoning model ("the database is a log").

**Cost.** Higher storage volume; consumers must learn to join on the pair, not on `block_number` alone.

**When to revisit.** Only if a future phase requires hard deletion for compliance reasons. Even then, the answer is more likely tiered archival than mutation.

---

## 2. Hexagonal beacon services (Go), not a monolithic indexer

**Decision.** Each beacon and each indexer is its own service with explicit ports and adapters. The data spine is composed of many small processes, not one big one.

**Considered.**
- *Single indexer process per environment*: simpler ops, single deploy.
- *Function-as-a-service per indexer*: lower operational footprint, no long-running processes.

**Why.** If one service breaks, the others keep running. It also keeps each beacon's permissions narrow — putting them all in one process would mean that process needs the combined permissions of every beacon, which is more authority than any single one should hold. Function-as-a-service didn't fit either: the watcher and stream consumers run continuously, not in short bursts.

**Cost.** More processes to operate. More deploy pipelines. Higher baseline node count.

**When to revisit.** If the marginal indexer ever becomes trivial enough that consolidation is cheaper than keeping it separate. 

---

## 3. Managed Kubernetes, not bare-metal or non-Kubernetes containers

**Decision.** All workloads run on a managed Kubernetes service.

**Considered.**
- *Bare-metal with custom orchestration*: maximum control, lowest cost at scale.
- *Container-orchestrator-as-a-service (non-Kubernetes)*: simpler ops surface.

**Why.** The Phase 1 team is small. The platform must be operable by a small team and must accept new engineers without months of platform onboarding. Kubernetes is the only orchestration where the labour pool, ecosystem (GitOps, autoscaling, observability), and managed offerings all meet that bar simultaneously.

**Cost.** Standard Kubernetes complexity tax. Managed-service constraints (limited control plane, vendor-set version cadence).

**When to revisit.** If team size or scale grows to where the managed contract becomes a meaningful ceiling.

---

## 4. ARM64 nodes, spot-first / on-demand fallback

**Decision.** Compute defaults to ARM64 architecture and spot capacity, with on-demand fallback for shortfalls.

**Considered.**
- *x86_64-only*: broader software compatibility.
- *On-demand-only*: predictable capacity, no reclamation events.

**Why.** Cost. ARM64 + spot dominates the price curve for the workloads Sky runs (Go beacons, indexers, Python source-of-truth). The architectural cost (spot reclamation) is absorbed by the patterns already required for resilience — stateless services, idempotent boundaries, replay-safe consumers.

**Cost.** Some software (rare) is x86-only. Spot reclamation is part of normal operations; consumers must be resilient to mid-shift restart.

**When to revisit.** If a critical workload is x86-only and cannot be ported. Pin per-workload, not globally.

---

## 5. Hub-and-spoke GitOps, not per-cluster controllers

**Decision.** A single hub cluster runs the GitOps reconciler and reconciles desired state into both clusters.

**Considered.**
- *Per-cluster controllers*: each cluster runs its own reconciler against the same Git repo.
- *Push-based deploy from CI*: CI pipelines apply directly to clusters.

**Why.** A single hub is the simplest mental model: one place to look for "what should be running, where." Per-cluster controllers double the operational surface and double the failure modes. Push-from-CI breaks the GitOps invariant ("Git is the desired state") and creates non-reproducible deploys.

**Cost.** The hub is a single point of operational dependency for change propagation (not for runtime). If the hub is down, running services keep running; deploys pause.

**When to revisit.** If hub-driven reconciliation becomes a bottleneck — likely only at significantly larger scale than Phase 1.

---

## 6. PostgreSQL with TimescaleDB extension, not bespoke time-series stores

**Decision.** Block-derived and oracle-derived data lives in a managed PostgreSQL with the TimescaleDB extension.

**Considered.**
- *Bespoke columnar time-series database*: better compression and query performance for time-series.
- *Cloud-native data lake*: cheaper at-rest storage, slower interactive queries.

**Why.** PostgreSQL's relational model fits the protocol's joins (positions × prices × allocations). The TimescaleDB extension brings time-series ergonomics (hypertables, continuous aggregates) into that relational model. A pure columnar store would have forced denormalisation; a data lake would have compromised interactive querying for indexers and dashboards.

**Cost.** Storage cost vs pure columnar. Some operational complexity from running an extension on managed Postgres.

**When to revisit.** If a sufficiently demanding analytical workload needs columnar performance. Then the answer is probably to *add* a columnar warehouse alongside, not to replace Postgres.

---

## 7. FIFO message stream as the cross-service backbone

**Decision.** Inter-service communication for block events uses a FIFO message stream with deduplication.

**Considered.**
- *HTTP synchronous calls between services*: simpler dev story.
- *Pub/sub without ordering guarantees*: cheaper, more scalable.
- *Direct database polling*: no message-stream component.

**Why.** Ordering and idempotent replay are non-negotiable. Synchronous HTTP couples capacity and failure. Pub/sub without ordering breaks per-block consumer logic. Direct DB polling is intractable at the cross-domain boundary.

**Cost.** FIFO throughput is lower than non-FIFO. Operationally one more component.

**When to revisit.** If a domain emerges where ordering doesn't matter; that domain can use a separate non-FIFO stream. Don't reuse a single backbone for both ordered and unordered traffic.

---

## 8. Branch-aware graph database for Synome, not a relational schema

**Decision.** Synome stores identities, formulas, and dependencies in a branch-aware graph database, with Git as the durable source for specs.

**Considered.**
- *Relational schema*: familiar, well-tooled.
- *Document store*: simpler write model.
- *Pure Git, no DB*: maximally simple.

**Why.** The data model is fundamentally a directed graph (formulas reference other formulas; identities depend on other identities). Branch-awareness is essential because the Atlas evolves on branches that must be comparable and roll-backable. A relational schema would force graph-shaped queries through joins and lose branch ergonomics. Pure-Git would force application-side traversal of every formula on every read.

**Cost.** Smaller ecosystem than relational DBs. More specialised operational knowledge.

**When to revisit.** If branch-awareness or graph queries cease to be load-bearing. Currently both are central; this is unlikely.

---

## 9. Identity-aware mesh for control-plane access, not bastion hosts

**Decision.** Cluster admin, GitOps UI, and out-of-band tooling are reachable only through a WireGuard-based identity-aware mesh.

**Considered.**
- *SSH bastion hosts*: traditional, well-understood.
- *VPN with shared credentials*: simple to operate.
- *Public exposure with strong authentication*: simplest from the user side.

**Why.** Mesh networks bind access to identity, not to a network position. There are no shared credentials. The mesh integrates cleanly with the SSO that already gates human identity. Bastions add a long-lived target with broad reachability; shared-credential VPNs scale poorly with team membership.

**Cost.** Every operator runs a mesh client. The mesh itself is one more component to keep healthy.

**When to revisit.** If a future SSO-native mesh-less alternative reaches parity. Not currently visible.

---

## How to add a new entry

When making a new architecture decision, write an entry in this style:

1. Phrase the decision in one sentence.
2. List at least two realistic alternatives. "We did this because there was no other option" is almost never true; if it is, say so explicitly.
3. State the *principle* that decided it, not just "it's better." A trade-off is two answers; the entry should say which property the principle prioritises.
4. Be honest about the cost. A decision with no cost is rarely a real decision.
5. Name the signal that would change the answer.

---

## Cross-references

- `decisions/constraints-and-assumptions.md` — the boundary conditions inside which these trade-offs were made
- `reference/technology-stack-matrix.md` — what each decision picked at the per-layer level
- `reference/security-governance-model.md` — protocol-side decisions about authority and access
- `operational/resilience-patterns.md` — how these decisions show up under stress
