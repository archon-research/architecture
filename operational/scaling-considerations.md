# Scaling Considerations

How each layer of the Phase 1 architecture scales, where the natural ceilings are, and how growth is absorbed. Conceptual; specific instance types, throughputs, and cluster sizes are deliberately omitted.

## What we're scaling against

Phase 1 growth comes from four largely-independent vectors:

| Vector | What grows | Felt by |
|---|---|---|
| **Block volume** | Ethereum (and L2) block rate, transaction count, log density | Watcher, Backfill, Backup, position trackers |
| **Protocol scope** | Number of source protocols indexed, oracle count | Position trackers, oracle worker, time-series database |
| **Synomic Agent count** | Number of Primes, Halos, Generators registered | Beacons, Synome platform, on-chain authority |
| **Activity rate** | Operations per Synomic Agent per day | Beacon services, message stream, settlement cadence (Phase 3+) |

A single growth event usually loads one of these vectors, not all four. That separation is what allows the architecture to scale incrementally.

---

## Per-layer scaling behaviour

### Watcher / data ingest

**Scaling characteristic:** essentially fixed work per block. The watcher does not get harder as more agents are onboarded; it gets harder as the chain produces more (and denser) blocks.

**Headroom:**
- Throughput is bounded by the upstream RPC provider and by the time-series database's write capacity, not by application code.
- Backfill is independently scaled and intentionally slower than live ingest; it absorbs gaps without competing for the same connection.
- Backup Worker is asynchronous; archive lag has no effect on live operation.

**Where pressure shows up first:** time-series database writes during high-traffic blocks. The path forward is the standard time-series-DB toolkit — chunking, hypertables, partitioning by recent vs. historical — applied to a managed product, not bespoke sharding.

### Position trackers and oracle worker

**Scaling characteristic:** linear in the number of source protocols and oracles. Each source has its own indexer process; adding a source adds a process.

**Headroom:**
- Indexers are independent. A slow new tracker cannot block existing ones.
- Each tracker reads from the same FIFO event stream; the stream's per-consumer-group semantics let consumers fan out without coordinating.
- Replay is safe (idempotent writes), so a tracker can be re-deployed and re-run without incident.

**Where pressure shows up first:** the cumulative write load on the time-series database as the number of trackers grows. The mitigation is the same as for the watcher — managed time-series partitioning. A second mitigation is per-tracker batching where the source allows.

### Platform (Kubernetes, GitOps, observability)

**Scaling characteristic:** the standard managed-Kubernetes contract. Workload-bound, with autoscaling at the node and pod level.

**Headroom:**
- Node autoscaling is spot-first / on-demand fallback; capacity for new workloads is acquired on demand.
- Hub-and-spoke GitOps means adding a new namespace or workload is a Git change, not a cluster reshape.
- Observability is centrally aggregated. The bottleneck moves into the observability platform, which is independently scaled.

**Where pressure shows up first:** ingestion volume into observability when many trackers come online at once. The standard toolkit (sampling, aggregation, retention tiering) applies.

---

## Architectural amplifiers and dampeners

Some patterns make growth easier. Some make it harder. Naming them helps when reviewing future designs:

### Amplifiers (good)

- **Stateless beacons.** Capacity is just more replicas.
- **Idempotent boundaries.** Retries, replays, and failovers compose cleanly.
- **Append-only data spine.** New consumers cost compute, not coordination.
- **Authority envelopes as rate caps.** Capacity ceilings are explicit and known up front.

### Dampeners (avoid)

- **Cross-service synchronous calls between beacons.** Couples capacity planning. Prefer asynchronous via the FIFO stream.
- **In-place mutation of historical state.** Breaks replay, audit, and reorg handling.

---

## Cross-references

- `operational/resilience-patterns.md` — how the same components behave under partial failure
- `reference/technology-stack-matrix.md` — the per-layer technology choices that determine scaling characteristics
- `roadmap/laniakea-phase-evolution.md` — the factory and automation phases that shift the scaling story
