# Synome Concepts for Engineers

A working translation of the Synome / Atlas / Beacon vocabulary into the language an engineer uses every day. Read this once and the rest of `laniakea-docs` becomes much more parseable.

---

## The one-paragraph version

The **Atlas** is the protocol's constitution — the rules the system must obey. The **Synome** is the data and code platform that stores the Atlas in a machine-readable form (specs in restricted Python, parsed into symbolic math, stored in a branch-aware graph database). A **Synomic Agent** is a durable on-chain entity that owns capital and can act. A **Beacon** is the bounded action aperture through which an agent (human or automated) actually does anything in the world. **BEAMs** are the on-chain authority tokens that grant beacons high-authority operations, organised in a strict hierarchy (admin → configurator → process). **Sentinels** are a distinguished, high-authority class of beacon that watch for misbehaviour and can halt the system. Phase 1 builds the foundation; Sentinels and most automation arrive in Phase 9–10.

That paragraph contains every term you need to read the rest of this repo.

---

## The vocabulary, mapped to engineering ideas

| Synome term | Engineering analogue | Notes |
|---|---|---|
| Atlas | Spec / constitution / invariant set | Not code yet — written as Python specs and reviewed by humans |
| Synome | Source-of-truth platform | Like a CMS for the constitution; stores specs, formulas, dependencies |
| Synomic Agent | On-chain account + treasury + governance | An identity that can own and act |
| Beacon | The single API surface of an agent | Every action an agent takes goes through a beacon |
| Authority Envelope | A typed authorization token: "what this beacon may do" | Rate limits, allowed targets, attestation duties |
| BEAM | An on-chain capability granted to a beacon | Power level depends on which BEAM (a > c > p) |
| Teleonome | A private AI that is *not* a beacon | "Dark by default" — invisible to the protocol unless it runs a beacon |
| Sentinel | A high-power, high-authority watchdog | Phase 9–10 only |
| Crystallization | "Promote this evidence to a binding rule" | The path by which observation becomes constitution |

A handy mental shorthand: **Atlas = the spec**, **Synome = the spec store**, **Beacon = the API endpoint**, **BEAM = the IAM grant**, **Sentinel = the circuit breaker**.

---

## How the pieces fit at runtime

```
   Atlas (the rules)
       │
       │  encoded as
       ▼
   Synome  ──►  specs, formulas, agent registry, beacon registry
       │
       │  reads (off-chain)              │  reads (on-chain via codegen / config)
       ▼                                 ▼
   Beacon services                   Smart contracts (PAU, Configurator, BEAMs)
       │   (LPLA / LPHA / HPHA)             │
       │   validate + execute               │  enforce authority bounds
       ▼                                    ▼
              ──── Public chain state ────
                       ▲
                       │ observes, attests, eventually reacts
                       │
                Sentinels (Phase 9–10)
```

The pattern is the same one engineers use every day:

- *Spec* in a single source of truth.
- *Codegen and config* push that spec into the runtime (services and contracts).
- *Enforcement* happens at well-defined boundaries (BEAMs on-chain, validation off-chain).
- *Observability* feeds back evidence — and in later phases that evidence can crystallize into new spec.

---

## Beacons, in concrete terms

A Phase 1 beacon is, in implementation terms, a service plus an on-chain role plus a Synome registration:

- **Service**: a Go or Python process running on the platform, with its own deployment, secrets, and observability. Lives next to other beacons in the same cluster.
- **On-chain role**: a contract address holding one or more BEAMs (cBEAM and/or pBEAM). The role bounds what the service can do regardless of what the service tries to do.
- **Synome registration**: a record in the source-of-truth platform that names the beacon, its authority envelope, the agent it is anchored to, and the attestation obligations it owes.

If any of those three is missing, the beacon does not exist as far as the protocol is concerned.

### The five non-negotiables

Every beacon, regardless of class, must be:

1. **Registered** in Synome
2. **Scoped** by an explicit authority envelope
3. **Anchored** to one Synomic Agent
4. **Observable** — every action emits enough signal for a Sentinel (current or future) to verify it
5. **Revocable** — its authority can be removed without redeploying the protocol

This is the security contract. Engineering reviews of new beacons should check all five.

### The 2×2 matrix

|   | **Low Authority** | **High Authority** |
|---|---|---|
| **Low Power** | LPLA — read-only verification, simple data exchange. (`lpla-verify`) | LPHA — Keepers: deterministic rule executors. (`lpha-relay`, `lpha-nfat`, `lpha-attest`, `lpha-council`) |
| **High Power** | HPLA — sophisticated peer-to-peer trading own capital. | HPHA — Sentinels and high-authority governance. (Phase 9–10) |

*Power* describes the sophistication of the algorithm (a teleonome behind the beacon). *Authority* describes how much it can affect protocol state. Phase 1 deploys only LPLA and LPHA — the lower row is deferred.

---

## What makes the BEAM hierarchy worth understanding

Every authority an off-chain service has on-chain comes from a BEAM. There are exactly three:

- **aBEAM (Admin)** — registers PAUs, grants cBEAMs. Held only by Council. Adds wait 14 days.
- **cBEAM (Configurator)** — sets rate limits, onboards approved targets. Granted to Relay Beacons.
- **pBEAM (Process)** — executes transactions inside the bounds cBEAM has set.

The crucial property:

> **Day-to-day execution rides on the lowest authority. The highest authority is the slowest and rarest.**

A compromised executor (`pBEAM` holder) cannot expand its own envelope. To do that it would need cBEAM, which only Council grants, and only after the 14-day timelock during which Council can revoke. This is a deliberate inversion of "the most active component holds the most power."

When you see a Phase 1 beacon name like `lpha-relay`, what you should picture: a service that holds **cBEAM and pBEAM** (it can both configure rate limits and execute), but whose authority was granted by an `lpha-council` agent that holds **aBEAM**.

---

## Atlas, specs, and crystallization

The Atlas is *not* application code. It is the constraint set the application code must satisfy. Storing it in machine-readable form is what makes Synome useful:

- Specs are written in a restricted Python subset (no classes, no lambdas, no comprehensions). This restriction makes them statically analysable and convertible to symbolic math.
- A `flake8` plugin enforces the subset at edit time.
- Specs are parsed into symbolic-math expressions, which can render to LaTeX for humans (Explorer UI) or to executable Python for runtimes.
- Specs live on branches in a graph database. Reads are branch-scoped; you can compare branches the way you would compare Git branches.

**Crystallization** is the term for the path: probabilistic evidence → reviewed observation → binding spec. Phase 1 does not automate this. Phase 9+ may. For now, treat it as a well-defined human review process that updates the Atlas.

---

## Mental models that will save you time

A few framings that have proven useful when reading or designing in this system:

1. **Beacons are the only API.** If a service is not a beacon, it cannot affect protocol state. If a beacon is not registered, it cannot affect protocol state. Most architectural questions reduce to "which beacon owns this responsibility?"

2. **High Power ≠ High Authority.** A clever model running a trading strategy with its own capital is High Power but Low Authority — it cannot move protocol funds. A boring deterministic rate-limit checker is Low Power but High Authority. The protocol cares mostly about authority.

3. **The mapping document is the Rosetta stone.** When a doc switches between "Relay Beacon" and "API gateway", `mappings/laniakea-engineering-business-mapping.md` will tell you they are the same thing.

---

## Cross-references

- `reference/glossary.md` — every term used above, defined in one place
- `reference/security-governance-model.md` — how authority is granted, enforced, and revoked
- `mappings/laniakea-engineering-business-mapping.md` — engineering ↔ business term mapping for Phase 1 components
- `roadmap/laniakea-phase-evolution.md` — when Sentinels, Crystallization automation, and the rest arrive
