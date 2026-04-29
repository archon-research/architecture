# Phase 1 Status

A high-level architectural status of Laniakea Phase 1 (Foundation, Phases 0–4). This is a living view; consult the engineering repos' issue trackers for ticket-level detail.

This document deliberately stays at the level of capabilities and components. Names of specific environments, accounts, regions, and vendor products are not included.

---

## Headline status

| Era | Status |
|---|---|
| **Phase 0** — Sky reorganisation, Atlas v0 | Complete |
| **Phase 1** — Synome-MVP, Beacon framework, PAU pattern, data spine | **In progress (the active focus)** |
| **Phase 2** — LCTS, srUSDS, NFAT | Partially built; rolling in |
| **Phase 3** — Daily Settlement Cycle | Designed; not yet active |
| **Phase 4** — Configurator Unit fully replaces spells; ALDM/Lindy/SPTP active | Configurator Unit operational; full Phase-4 set in progress |

The system is operating in production with Phase 1 capabilities. Phases 2–4 are landing incrementally rather than all at once.

---

## What is built (Phase 1 core)

### The institutional layer
- **Synomic Agent registry** is live in Synome.
- **PAU pattern** (Diamond) is the standard control surface. Existing Primes have been migrated to it.
- **aBEAM / cBEAM / pBEAM hierarchy** is operational, with the 14-day timelock on aBEAM additions.
- **Council Beacon** (`lpha-council`) holds aBEAM and is the only path to register PAUs and grant cBEAMs.

### Beacons in service
- **`lpla-verify`** — read-only state verification.
- **`lpha-relay`** — Relay Beacon with cBEAM + pBEAM. Rate-limited, target-bounded entry aperture.
- **`lpha-nfat`** — NFAT (bespoke Term-deal) execution.
- **`lpha-attest`** — produces signed attestations consumed by other beacons and audit.
- **`lpha-council`** — Council Beacon (aBEAM holder).

### The data spine
- **STL block-data service** (Go) running in production.
- **Watcher / Backfill / Backup Worker** triad in place: live indexing, gap-filling, durable archive.
- **Per-protocol position trackers** writing into a managed time-series database. New trackers added as new sources come online.
- **Oracle price worker** writing snapshots into the same time-series database.
- **Append-only convention** with block versioning enforced everywhere.

### Source of truth
- **Synome platform** (Python + branch-aware graph database) live.
- **Atlas spec submodule** under continuous editorial work; specs validated by the `flake8` plugin and rendered through the symbolic-math representation.
- **Explorer UI** available for browsing identities, formulas, dependencies, and branch comparison.

### Platform
- **Two managed Kubernetes clusters** (production + staging) with shape-equivalent topology.
- **Hub-and-spoke GitOps** reconciling both clusters from a single Git repository.
- **App-of-Apps** bootstrap pattern enumerates per-team / per-environment workloads.
- **ARM64 node autoscaling** with spot-first / on-demand fallback.
- **Identity-aware mesh** for control-plane access; **no public exposure** of cluster admin endpoints.
- **Secret sync** from a managed secret store; **workload IAM** bound to service accounts.
- **Centralised observability** (metrics, traces, logs) with audit-stream alerting.
- **Read-only contractor proxy** for external auditors.

---

## What is in progress

### Phase 2 — capacity-constrained primitives
- **LCTS** (Liquidity Constrained Token Standard) — queue-based conversion mechanism. Initial Halo Class wired up; broader rollout ongoing.
- **`srUSDS`** issuance via LCTS — Generator-side integration in progress.
- **NFAT** primitive — live; expanding the buybox templates per Halo Class.

### Phase 3 — daily settlement
- **Active Window / Lock Window / Moment of Settlement** — designed. Enforcement at the `lpha-relay` level is on the path; not yet activated in production.
- **Tug-of-war** allocation — modelled; not yet operational.

### Phase 4 — configurator and risk machinery
- **Configurator Unit** (BEAMTimeLock → BEAMState → Configurator) — deployed for spell-less Prime config.
- **ALDM, Lindy Duration Model, SPTP** — encoded in specs; integration into the live risk path in progress.

### Engineering hardening
- **Backfill / position-tracker resiliency** — error-handling and idempotency improvements ongoing.
- **Cross-environment parity** — closing remaining gaps so staging is a faithful pre-production gate.
- **Runbook coverage** — extending operational runbooks to match every beacon and indexer.

---

## What is explicitly *not* in Phase 1

These belong to later phases. Mentioning them in a Phase-1 design is a sign the design is reaching beyond scope.

- **HPHA Sentinels** (`stl-base`, `stl-stream`, `stl-warden`, `stl-principal`) — Phase 9–10.
- **Auction-based OSRC pricing** — post-`stl-base`.
- **Prime / Halo / Generator factories** — Phase 5–7.
- **Trading Halos** (AMM-backed) — Phase 6.
- **TEJRC / TISRC tokenisation as standard primitives** — Phase 8.
- **Probabilistic mesh and crystallization automation** — Phase 10.
- **Identity Networks, Skychain** — out of scope for Foundation.

---

## How to read this document

- "**Built**" means: deployed in production, observable, owned by a named team, with a runbook.
- "**In progress**" means: design exists, partial implementation, behind a feature flag or a single-Halo pilot.
- "**Not in Phase 1**" means: explicitly deferred. If a design appears to require it, that design is either premature or out of scope.

For ticket-level detail, follow the engineering repos:
- `~/workspace/stl` — the data spine and Phase-1 beacons
- `~/workspace/synome` — the source-of-truth platform
- `~/workspace/infrastructure` — the platform layer

---

## Cross-references

- `roadmap/laniakea-phase-evolution.md` — how Phases 5–10 build on Phase 1
- `reference/synome-concepts-for-engineers.md` — the vocabulary
- `reference/security-governance-model.md` — the authority model the beacons enforce
- `mappings/synome-engineering-business-mapping.md` — components in both vocabularies
