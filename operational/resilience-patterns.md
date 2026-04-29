# Resilience Patterns

How the Phase 1 architecture behaves under partial failure, and the patterns that make it recoverable. Conceptual; specific endpoints, regions, and product names are deliberately omitted.

## Design principles

Three principles drive every resilience choice in Phase 1:

1. **The data spine is append-only and reconstructable.** No failure mode should require deleting historical state. Every disagreement between sources is reconciled as a new versioned row, not as an in-place correction.
2. **Authority is the slowest thing.** High-authority operations (aBEAM grants, spec changes) move on the order of days. Low-authority operations (`pBEAM` execution) move on the order of seconds. A failure of the slow path does not block the fast path; a failure of the fast path cannot escalate into a slow-path action.
3. **Sentinels are the bound on damage.** Today the bound is human review and rate limits; in Phase 9–10 it is automated. The architecture is shaped so that adding Sentinels does not require redesigning the failure model — it tightens an already-existing bound.

---

## Per-component failure model

### Watcher (live block ingest)

**Failure modes:** RPC provider outage; WebSocket disconnect; node restart; chain reorg.

**Pattern:**
- Live ingestion runs continuously; on disconnect the service reconnects with backoff.
- A separate **Backfill** service polls via HTTP and fills any gap when the watcher resumes. The two services share an idempotent write path (`ON CONFLICT DO NOTHING`).
- Reorgs are not destructive. The service detects them and writes new rows under an incremented `block_version`, leaving prior versions intact.
- A **Backup Worker** continuously archives raw block data to durable object storage. Even a catastrophic loss of the operational database can be reconstructed from the archive.

**Result:** the data spine has three layers of defence — live, backfill, archive — and never overwrites history.

### Position trackers and oracle workers

**Failure modes:** consumer crash; bug in indexing logic; downstream database unavailable.

**Pattern:**
- Each consumer reads from a FIFO message stream with an explicit checkpoint.
- Restart resumes from the last checkpoint; the message stream's ordering and idempotent writes mean replay is safe.
- A bug in indexing logic is corrected by deploying a fix and replaying from a chosen point. Since writes are append-only, replay produces a corrected version next to the bad one rather than mutating the bad one.

**Result:** every indexing bug has a clean recovery path that does not require database surgery.

### Beacon services (`lpla-*`, `lpha-*`)

**Failure modes:** service crash; bad deploy; misconfigured authority envelope; loss of secret access.

**Pattern:**
- Beacons are stateless at the service level; all durable state lives in the data spine and on-chain.
- Multiple replicas behind the platform's standard load balancing.
- Authority bounds are enforced *on-chain* by the BEAM hierarchy. A misbehaving service cannot exceed its envelope, even if the off-chain validation is bypassed.
- A bad deploy is rolled back by the GitOps reconciler reverting to the previous commit. There is no out-of-band "fix in place".

**Result:** the worst a Phase-1 beacon can do is what its `cBEAM` configuration already permits — and that configuration was set by Council with a 14-day timelock.

### Council Beacon (`lpha-council`)

**Failure modes:** key compromise (catastrophic); operational error in registering or revoking authority.

**Pattern:**
- aBEAM additions are subject to a 14-day timelock, giving the Core Council a window to revoke a malicious or mistaken proposal before it activates.
- Council key access is bound to multi-party human approval. There is no automation that can grant aBEAM.
- Council action is logged on-chain and into the audit stream; an unexpected Council action is the highest-priority alert in the system.

**Result:** even if everything else fails, the timelock is the last line of defence.

### Source-of-truth platform (Synome)

**Failure modes:** graph database unavailable; bad spec merged; branch divergence.

**Pattern:**
- Specs are versioned in Git via the spec submodule. The graph database is the *active* representation; Git is the *durable* one. Loss of the graph DB is recoverable from Git.
- A bad spec is caught at three stages: editor (`flake8` plugin), CI (full validator), code review. Promotion of a spec to the main branch goes through human review.
- Branches in the graph database are first-class. Comparison and rollback are cheap.

**Result:** spec mistakes are reversible at the spec level; runtime services never silently follow a contested spec.

### Platform (managed Kubernetes, GitOps, observability)

**Failure modes:** cluster-wide outage; GitOps drift; secret-store outage; node-pool exhaustion.

**Pattern:**
- Two clusters (production + staging) of equivalent shape. Staging is a faithful pre-production gate.
- Hub-and-spoke GitOps: a single hub reconciles both clusters. Drift is detected and reverted automatically; out-of-band changes (`kubectl apply`, console clicks) are an incident, not a workflow.
- Secret access is workload-scoped via cloud IAM bound to service accounts. A single secret-store outage does not invalidate already-issued credentials.
- Node autoscaling is spot-first with on-demand fallback. Spot reclamation triggers replacement, not service loss.
- Observability data is aggregated centrally with immutable retention so that an in-cluster failure does not blind the response.

**Result:** the platform's failure modes are bounded by the standard managed-Kubernetes contract; novel failure modes specific to Sky are concentrated in the application layer above.

---

## Cross-references

- `operational/scaling-considerations.md` — load behaviour and bottlenecks
- `reference/security-governance-model.md` — how authority is granted, enforced, and revoked
- `reference/synome-concepts-for-engineers.md` — vocabulary
- `roadmap/laniakea-phase-evolution.md` — when Sentinels and the OSRC bound become operational
