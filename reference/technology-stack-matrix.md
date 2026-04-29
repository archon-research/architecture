# Technology Stack Matrix

What technology is in use at each layer of Laniakea Phase 1, and why it was chosen. This is a **snapshot of Phase 1**; the stack evolves with each phase — see `roadmap/laniakea-phase-evolution.md`.

This document deliberately stays at the conceptual level — concrete account, environment, network, and resource identifiers live in the engineering repos themselves and are not duplicated here.

---

## Stack at a glance

| Layer | Technology | Repo | Phase introduced |
|---|---|---|---|
| Cloud platform | AWS, single primary region | `infrastructure` | Phase 0 |
| Compute | Managed Kubernetes (EKS) on ARM64 (Graviton) | `infrastructure` | Phase 1 (migrating from container-on-Fargate) |
| Node autoscaling | Karpenter | `infrastructure` (gitops) | Phase 1 |
| GitOps | ArgoCD, hub-and-spoke across two clusters | `infrastructure` (gitops) | Phase 1 |
| IaC | OpenTofu (Terraform-compatible) | `infrastructure` | Phase 0 |
| Block-data ingestion | Go 1.26+ services | `stl/stl-verify` | Phase 1 |
| Service architecture | Hexagonal (ports & adapters) | `stl/stl-verify` | Phase 1 |
| OLTP / time-series DB | PostgreSQL + TimescaleDB (managed) | `infrastructure`, `stl/stl-verify/db/migrations` | Phase 1 |
| Cache / ephemeral state | Managed Redis | `infrastructure`, used by `stl-verify` | Phase 1 |
| Object storage / archive | AWS S3 | `infrastructure`, `stl/stl-verify/cmd/raw_data_backup` | Phase 1 |
| Event distribution | SNS FIFO + SQS FIFO | `infrastructure`, `stl/stl-verify` | Phase 1 |
| Workflow orchestration | Temporal (self-hosted) | `infrastructure`, `stl/stl-verify/cmd/temporal-worker` | Phase 1 |
| Source of truth (the Atlas) | Python (uv) + TerminusDB + SymPy | `synome`, `synome/next-gen-atlas` | Phase 1 |
| Observability | OpenTelemetry → managed Grafana Cloud (Logs / Traces / Metrics) | `infrastructure` (per ADR-004) | Phase 1 (rollout in progress) |
| Contractor DB access | Connection-pool proxy on bastion → managed-DB read replica | `infrastructure` (per ADR-002) | Phase 1 |
| Container registry | AWS ECR | `infrastructure` | Phase 1 |

---

## Why these choices — by layer

### Compute: managed Kubernetes on ARM64

**Choice.** EKS with Karpenter; ARM64 (Graviton) node families; spot-first with on-demand fallback. Migrated from a container-on-Fargate setup (ADR-001).

**Why.**
- ARM64 is materially cheaper for steady-state compute and aligns with the `stl-verify` Dockerfiles, which build for `linux/arm64`.
- Karpenter consolidates pods aggressively and avoids over-provisioned managed node groups.
- Kubernetes (vs. Fargate) gives us first-class sidecars, IRSA for cloud auth, and a single substrate for both the Go services (`stl`) and the Python platform (`synome`).
- A hub-and-spoke ArgoCD model means **one** install governs all clusters; the spoke cluster doesn't run its own ArgoCD.

**Trade-off.** Cluster egress is restrictive by default. New cross-VPC connectivity (e.g., a new managed DB) requires explicit egress + VPC peering. See `decisions/trade-offs.md`.

### Data: PostgreSQL + TimescaleDB (managed)

**Choice.** A managed PostgreSQL service with the TimescaleDB extension. Hypertables for time-series data (block state, prices, positions). Reached from EKS via VPC peering.

**Why.**
- All of `stl-verify`'s state is naturally time-series indexed by `(block_number, block_version)`. Hypertables with chunking and compression handle the volume cleanly.
- The managed service handles backups, replication, and PITR for us; we don't operate database servers ourselves.
- A read replica with a SELECT-only PG role gives us a clean, audited perimeter for external contractors without exposing the primary.
- PostgreSQL is the natural fit for the relational data the per-protocol indexers write.

**Append-only convention.** Price tables use `ON CONFLICT DO NOTHING` and are never updated. Block reorgs are handled by inserting a new `(block_number, block_version+1)` row, not by deleting earlier rows. This is required for the **crystallization invariant** — once an attestation is written, it cannot be silently rewritten.

### Caching: managed Redis

**Choice.** Managed Redis, reached from EKS via VPC peering and a tightly scoped security-group rule. Two-day TTL on cached block data, keyed by `stl:{chainId}:{blockNumber}:{version}:{dataType}`.

**Why.** The watcher needs sub-millisecond reads of recent block data while it processes downstream consequences. The TTL is the smallest cache that absorbs hot reads while keeping cold reads database-bound.

### Eventing: SNS FIFO + SQS FIFO

**Choice.** All block events fan out via a single SNS FIFO topic; each indexer consumes a dedicated SQS FIFO subscription with a per-chain message-group-id.

**Why.** FIFO with a per-chain group preserves per-chain ordering — necessary for any indexer that has to apply state transitions in order. SNS+SQS is the cheapest at-least-once delivery option that doesn't require us to operate Kafka. ADR-0002 in `stl/docs/adr/` evaluated five options (Redis Streams, SNS/SQS, NATS, PostgreSQL, Kafka) and chose SNS/SQS for the simpler operational footprint.

### Workflow: Temporal

**Choice.** Self-hosted Temporal with workers polling a shared task queue.

**Why.** Two scheduled jobs need durable, retryable orchestration: a periodic price fetch and a less-frequent data-validation cross-check. Temporal also gives us the substrate for the NFAT process orchestration described in `stl/docs/adr/0002-nfat-process-orchestration-architecture.md` (state-machine + event-bus pattern).

### Source of truth: Python + TerminusDB

**Choice.** Synome stores the Atlas as Python specs (restricted subset), validated by a custom `flake8` plugin, parsed into SymPy formulas, and persisted into a TerminusDB graph database. Two repos: `synome` (platform) and `next-gen-atlas` (specs, vendored as a submodule and mirrored from upstream).

**Why.**
- **Python specs** make the Atlas authorable by the same people who write the risk and accounting logic. The restricted subset (no classes, lambdas, comprehensions) keeps every spec parseable into a math object.
- **SymPy** as the canonical math representation lets us render to LaTeX (for the whitepaper / UI) and to executable Python (for runtime evaluation) from the same source.
- **TerminusDB** is branch-aware and Git-backed — first-class branching of the Atlas itself. A proposed governance change can be developed on a branch, compared against `main`, and merged through review.
- The `flake8` plugin turns spec validation into editor diagnostics — authors see violations as squiggles in the IDE, not at parse time.

### Frontend: Synome Explorer UI

The Explorer UI is the human-facing surface on the source of truth: identity browsing, formula rendering, branch selection, history, and identity-dependency graphs. All identity reads are branch-scoped via a `?branch=<branch_name>` query parameter.

### Observability: managed Grafana Cloud + OTel

**Choice.** Per ADR-004, all telemetry (Logs, Traces, Metrics) flows via the OTel Collector to a managed Grafana Cloud stack — one stack per environment.

**Why.** A managed observability stack is materially cheaper than running our own Prometheus + Loki + Tempo at this scale. ESO + Secrets Manager handles OTel-token rotation cleanly.
