# Constraints & Assumptions

The boundary conditions Phase 1 was designed inside. If any of these change materially, the relevant downstream decisions need revisiting.

This document is conceptual. It does not enumerate specific products, instance types, or operational targets — those live in the engineering repos.

---

## Hard constraints (cannot be violated by design)

These are properties of the protocol or its environment that the architecture must accept. Violating any of them is a protocol-level event, not a design choice.

### From the Atlas / governance

- **Every action that affects protocol state must go through a registered beacon.** No back doors, no out-of-band administrative paths.
- **Authority follows the BEAM hierarchy: aBEAM → cBEAM → pBEAM.** No service may grant itself broader authority than it already has.
- **High-authority changes (aBEAM additions) are subject to a 14-day timelock.** No emergency override.
- **Spec changes go through human review.** Crystallization automation is Phase 10.
- **Beacons are revocable.** A design that requires an irreversible authority grant is incompatible with the model.

### From the operating environment

- **Single primary region in Phase 1.** Multi-region is not in scope. Designs that require active-active geographic distribution are out of scope until the architecture is explicitly extended.
- **All workloads run on managed Kubernetes.** Bare-metal, serverless-only, or non-containerised workloads are not in the supported set.
- **All change to running systems goes through GitOps.** Out-of-band changes — applying manifests directly, editing resources by hand, clicking through a console — are not a supported workflow; the reconciler treats them as drift and reverts them.
- **Public exposure of cluster control planes is not permitted.** Access is through an identity-aware mesh.
- **No long-lived static credentials in workloads.** Workload identity is bound to cloud IAM via service accounts.

### From the data spine

- **Append-only writes.** `ON CONFLICT DO NOTHING` is the default; in-place mutation of historical state is prohibited.
- **Block versioning is the canonical key.** Joins on block-derived state must use `(block_number, block_version)`, not `block_number` alone.
- **Audit reconstruction must be exact.** Approximate or lossy archival is incompatible with the protocol's audit posture.

---

## Soft constraints (negotiable, but require explicit decision)

These are operational choices made for Phase 1. Changing them is a meaningful architectural decision and should be documented in `decisions/`.

| Constraint | Why this in Phase 1 | When it might change |
|---|---|---|
| **Two environments only** (production + staging) | Staging is shape-equivalent to production; it is the gate for change | Adding a third environment (e.g., chaos/perf) is a deliberate platform decision |
| **One time-series database per environment** | Single source of truth for protocol state, simpler operational model | Splitting into per-domain databases (e.g., one for positions, one for prices) becomes worth considering only in later phases, once data volumes or independent scaling needs justify the added operational cost |
| **One FIFO event stream as the cross-service backbone** | Ordering and idempotent replay are the load-bearing properties | A second stream becomes meaningful when domains diverge enough that ordering between them stops mattering |
| **Per-source-protocol indexer process** | Independent failure domains, simple replay | Consolidation is possible but rarely worth the coupling cost |
| **ARM64 nodes, spot-first** | Cost efficiency, broad availability | Reverting to on-demand-first is a cost decision, not a correctness one |
| **Centralised observability platform** | Single pane for metrics, traces, logs, audit | Splitting the audit stream is a security decision worth its own ADR |

---

## Assumptions (true today; revisit if they shift)

These are statements about the world that drive design choices. None is guaranteed for the long run; each is reasonable now.

### About data sources

- The chain remains the authoritative source for protocol state. Off-chain feeds (oracles, price providers) are validated against on-chain settlement where possible.
- Independent references for validation (e.g., a second indexer or a public block explorer) are available and treated as a check, not as a fallback.

### About operations

- All production access is identity-bound and auditable. No shared accounts.
- Contractor and external-auditor read access is narrowly scoped through a bastion-hosted proxy on a least-privileged role.

### About the team

- All meaningful changes go through code review on a Git branch before reaching either environment.
- Runbooks exist (or are being written) for every beacon, and platform component in active use.
- The teams operating the engineering repos coordinate via the architecture repo for cross-cutting concerns; Phase-specific tickets remain in the engineering repos.

---

## How to use this document

- **Reviewing a design:** check it against every entry in *Hard constraints*. Any violation needs explicit re-architecture, not a workaround.
- **Reviewing an ADR:** *Soft constraints* and *Assumptions* are the ones the ADR should cite. Hard constraints rarely need to be argued; they are the floor.
- **Onboarding:** read this once, then read `roadmap/phase-1-status.md` for what is built inside these constraints.

---

## Cross-references

- `decisions/trade-offs.md` — the major design decisions made inside these constraints
- `decisions/README.md` — index of architecture decisions
- `reference/security-governance-model.md` — the protocol-side enforcement of the hard governance constraints
- `roadmap/laniakea-phase-evolution.md` — when soft constraints change shape
