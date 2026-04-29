# Security & Governance Model

How governance constraints flow from the Atlas through to running services, and how engineering enforces them today.

This is the conceptual model. Specific cluster names, account identifiers, IP ranges, secret names, and vendor product names are intentionally **not** included here.

---

## Two layers of "governance"

Sky uses the word *governance* in two distinct senses. They meet, but it helps to keep them separate when reasoning about security.

| Layer | What it governs | Where it lives |
|---|---|---|
| **Protocol governance** | What Synomic Agents are allowed to do with capital. Encoded in the Atlas. Enforced on-chain via BEAMs and PAUs. | Smart contracts + Synome specs |
| **Platform governance** | Who can deploy code, change infra, read secrets, view production data. | IAM + GitOps + cluster RBAC |

A change to Atlas constraints (protocol governance) and a change to who can `kubectl exec` into a pod (platform governance) travel through completely different review paths, but both must be auditable, reversible, and bounded by least privilege.

---

## Protocol governance: how Atlas constraints reach running code

```
Atlas (constitution)
   │  encoded as
   ▼
Specs in Synome  ──────►  symbolic formulas
   │                         │
   │ codegen / runtime read  │ codegen
   ▼                         ▼
On-chain authority structure (BEAMs)   Off-chain enforcement (beacon services)
   │                                       │
   ▼                                       ▼
PAUs / Controllers / RateLimits       Validation, rate checks, attestation
```

### On-chain authority: the BEAM hierarchy

Authority on-chain is granted in only one way: through a BEAM (Bounded External Access Module). BEAMs come in three flavours that form a strict hierarchy:

1. **aBEAM (Admin)** — registers PAUs, grants cBEAMs to other beacons. Held only by a Council Beacon (HPHA). Adds are subject to a 14-day timelock so the Core Council has time to revoke a malicious or mistaken proposal before it activates.
2. **cBEAM (Configurator)** — sets rate limits, onboards approved targets. Held by Relay Beacons. Granted (and revocable) only by aBEAM.
3. **pBEAM (Process)** — executes operations. Held by the same Relay Beacons. Bounded by the configuration cBEAM has set.

Important property: **the highest-authority component (aBEAM/Council) is the lowest-frequency one.** Day-to-day execution rides on pBEAM, which is the least powerful. This means a compromise of an executor cannot grant itself more power; it can only do what its authority envelope already allowed.

### Off-chain enforcement: the beacon contract

Every beacon — whether implemented as a contract role, a Go service, or a Python worker — carries five non-negotiable properties:

- **Registered** — known to the Synome.
- **Scoped** — its authority envelope (rate limits, approved targets, attestation obligations) is explicit and machine-readable.
- **Anchored** — tied to a specific Synomic Agent.
- **Observable** — every action is logged in a way that the corresponding Sentinel can read.
- **Revocable** — its authority can be removed without redeploying the world.

A service that does not satisfy all five is not a beacon. It cannot operate a Synomic Agent.

### Beacon classes and what they can do

| Class | Power | Authority | Typical role |
|---|---|---|---|
| **LPLA** | Low | Low | Read-only verification, simple data exchange |
| **LPHA** | Low | High | "Keepers" — deterministic rule executors |
| **HPLA** | High | Low | Sophisticated peer-to-peer beacons trading own capital |
| **HPHA** | High | High | Sentinels and high-authority governance beacons |

Phase 1 deploys LPLA and LPHA beacons only. HPHA Sentinels arrive in Phase 9–10. HPLA is governed by accord agreements but does not require Atlas changes to onboard.

---

## Platform governance: how engineering changes reach production

The platform mirrors the protocol's defence-in-depth posture. There is no path from a developer's laptop to a production capital action; every step crosses an enforcement boundary.

```
Developer
  │  (code review, branch protection)
  ▼
Git repo (source of truth for desired state)
  │  (signed commits, required approvals)
  ▼
GitOps reconciler (hub cluster)
  │  (cluster RBAC, namespace boundaries)
  ▼
Production / Staging clusters
  │  (workload IAM, network policy)
```

### Identity and access

- **Human identity** flows through a single identity provider with SSO. There are no shared accounts. Cluster, cloud, and database access are all bound to that identity.
- **Workload identity** uses cloud-native IAM bound to Kubernetes service accounts. No long-lived static credentials live in pods or in Git.
- **Privilege is least-privilege by default.** The set of operations a service account can perform is enumerated explicitly; broad wildcards are rejected at review time.

### Secrets

- A managed secret store is the only source of secret values.
- A controller mirrors the necessary subset into the cluster. Secrets never appear in Git, in container images, or in shell history.
- Rotation is automated where the secret type supports it; manual rotation is documented and exercised.

### Network posture

- Cluster control planes and admin consoles are not reachable from the public internet.
- Service-to-service traffic inside the cluster is governed by namespace and pod-level network policy.
- External auditors and contractors who need read access to operational data go through a bastion-hosted, read-only proxy bound to a least-privileged database role. They never touch production cluster control planes.

### Change control

- All cluster and infrastructure changes are made by GitOps reconciliation against a Git repository. Out-of-band changes (`kubectl apply`, console clicks) are an incident, not a workflow.
- Production deploys go through staging first. The two environments are shape-equivalent so that a passing staging test is a meaningful signal.
- Image provenance is verified: only signed images from the project's own registries are admitted.
- Destructive cloud-API actions (delete cluster, delete database, rotate KMS key) require multi-party approval.

### Observability for security

- Audit logs from cloud APIs, the cluster API server, GitOps reconciler, and identity provider are aggregated into a central observability platform with immutable retention.
- Alerting on the audit stream covers: privilege escalation, unexpected human access to production, GitOps drift, image admission failures, and credential issuance to unfamiliar identities.

---

## What is in Phase 1 and what is not

In scope today:

- LPLA and LPHA beacons, with explicit authority envelopes
- aBEAM / cBEAM / pBEAM hierarchy with the 14-day Council timelock
- Identity-bound human and workload access; no shared credentials
- GitOps-only change flow into both environments
- Mesh-based private access to control planes; no public exposure
- Append-only audit data with reconstructable per-block state via block versioning

Out of scope until later phases:

- HPHA Sentinels and the automated shutdown behaviour they enable (Phase 9–10)
- Daily Settlement cadence and the `lpha-relay` enforcement of Active / Lock windows (Phase 3+)
- Full risk-capital arithmetic on-chain (TEJRC / TISRC / OSRC tokenisation)
- Auction-based OSRC pricing (post-`stl-base`)

The architecture is shaped so these additions extend the model rather than replace it.

---

## Cross-references

- `reference/glossary.md` — definitions for every term used here (Beacon, BEAM, PAU, Sentinel, etc.)
- `reference/synome-concepts-for-engineers.md` — how Atlas / Synome / beacons relate, in engineering terms
- `mappings/synome-engineering-business-mapping.md` — Phase 1 components in both vocabularies
- `roadmap/laniakea-phase-evolution.md` — how the model deepens through phases 5–10
