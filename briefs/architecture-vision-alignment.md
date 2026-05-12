# How the Laniakea Architecture Satisfies the Synome Vision

## Purpose

This document maps the current engineering architecture — what we're building and operating today — to the Synome vision described in the laniakea-docs. The goal is to show that every engineering choice is a deliberate, faithful implementation of a specific concept in the vision, not a deviation from it.

The architecture is Phase 1. It's simplified. But the simplifications are structural choices that preserve the invariants the vision requires, while deferring complexity that isn't load-bearing yet.

---

## The Architecture at a Glance

Six components, top to bottom:

```
Blockchain                  — the external world
Data Ingest                 — our eyes on the world
Synome (Rules + State)      — the canonical data layer
Core                        — the compute layer
Access & Routing            — the regulated gate
Applications                — how humans interact
```

Each of these maps directly to a concept in the Synome vision.

---

## Component-by-Component Mapping

### Blockchain → "The World"

In the five-layer architecture, Layer 5 (Embodied Agents) interacts with the World. The World is external reality — it's where actions land and evidence originates. The blockchain is our World: smart contracts, events, oracles. It's where the protocol's on-chain state lives — USDS, BEAMs, agent contracts, price feeds.

We don't control it. We observe it and act on it through regulated apertures.

### Data Ingest → "Evidence Flows Back from the World"

The vision describes a closed loop: the system acts on the world, and the world returns evidence. Data Ingest is the evidence channel. It watches blockchains, scrapes blocks, transactions, events, and oracle prices, and writes them into the Synome.

In the full vision, this role belongs to **endoscrapers** — chain-watching processes that independently verify what beacons report. Our STL block-data service (watcher, backfill workers, oracle price indexers, position trackers) is exactly this. It supports Chainlink, Chronicle, Redstone, and protocol-native oracles. It writes into State — the evidence partition.

The evidence loop is operational today. Evidence flows in continuously.

### Synome → The Canonical Data Layer (Atlas/Synome Separation Realized)

This is the most important mapping. The vision describes a separation between the Atlas (human-readable constitution) and the Synome (machine-readable operational system). Our architecture implements this through two partitions within a single logical layer:

**Rules** = the machine-readable Atlas. Governance-set configuration that changes at governance speed: risk formulas, settlement logic, agent definitions, authority chains, lifecycle rules, token rules, matching rules. This IS the deontic skeleton — the sparse, load-bearing, deterministic connections that hold unconditionally once set.

**State** = the evidence layer. Operational data that changes at operational speed: positions, prices, NFAT records, attestations, risk scores, settlement records, breach records, audit trail. This is where evidence accumulates — the raw material that the probabilistic mesh, in its full form, would carry with truth values and confidence.

The vision says the Atlas is embedded IN the Synome, not separate from it. Our architecture reflects this: Rules and State are two partitions of one logical system — the Synome. The Atlas lives inside the Synome as its constitutional core.

**Why this matters for the governance window:** The Rules partition is the vessel. It captures governance decisions in machine-readable form. While humans can still meaningfully participate in governance — while we're in regime (1) of the governance window — every rule encoded in the Rules partition is a value locked in. The architecture makes this structurally natural: governance acts, Rules updates, Core enforces. The window is being used.

### Core → Beacons (The Regulated Apertures)

The vision's most load-bearing principle: *"Intelligence lives privately; power enters the world only through regulated apertures."*

Core is where those apertures live. Core services are beacons — registered, scoped, observable, revocable processes that read Rules and State, apply business logic, and write derived state back. They are the only thing that can act.

The beacon classification maps directly:

| Vision concept | Phase 1 implementation |
|---|---|
| LPLA (Low Power, Low Authority) | Read-only verification services, data validation |
| LPHA (Low Power, High Authority) | Settlement, NFAT lifecycle, attestation workflows — deterministic rule executors |
| HPLA (High Power, Low Authority) | Deferred (Phase 9+) — sophisticated trading with own capital |
| HPHA (High Power, High Authority) | Deferred (Phase 9+) — sentinels, continuous real-time control |

Phase 1 deploys the bottom row of the 2x2 matrix. The architecture supports the full matrix without structural change — adding sentinels and high-power beacons is a matter of deploying new Core services, not rewriting the architecture.

**The crystallization interface lives here.** In the full vision, governance sits at the crystallization interface — consuming probabilistic evidence, producing deontic commitments. In Phase 1, Core services perform a simplified version: they read evidence (State), apply rules (Rules), and produce derived state (back to State). When patterns emerge — breaches accumulate, penalties prove insufficient, risk parameters need adjustment — governance acts and Rules updates. The crystallization loop is manual in Phase 1 but structurally present. The evidence is there. The rules are there. The interface between them is there. Automation comes in Phase 9+.

### Access & Routing → The Gate Primitive

The vision describes a gate at every trust boundary: signature verification, whitelist checks, permission lookups, audit logging. Every submission passes through the gate. Default-deny. The gate is the chokepoint where "power enters the world through regulated apertures" becomes concrete.

Access & Routing is the gate. It authenticates every request from Applications, enforces RBAC permissions, and routes: read requests go directly to State (no compute needed), write/action requests go to Core (business logic required). No request bypasses it.

In the full vision, the gate checks `(auth $beacon $verb $target)` — a flat 3-tuple lookup. Our RBAC model implements the same pattern: who is this caller, what are they trying to do, on what target. The authority chain (aBEAM > cBEAM > pBEAM) maps to our permission hierarchy.

### Applications → Operators (No Inherent Authority)

The vision is explicit: operators at Level 1 make requests. They don't carry authority themselves. A human clicking a button has no more structural authority than an AI agent submitting an API call. Both are constrained by the same beacons, the same Rules, the same audit trail.

Applications is this layer. UIs, APIs, integrations — they're external consumers. They can't reach State or Core directly. Everything passes through Access & Routing. Adding new applications (including, eventually, AI operators) requires no architectural change — just new clients at the edge.

---

## Structural Invariants Preserved

These are the non-negotiable principles from the vision and how the architecture preserves each one:

| Invariant | How it's preserved |
|---|---|
| Intelligence lives privately; power through regulated apertures | Core services are the only writers. Access & Routing gates everything. Applications have no direct access. |
| The governance window is being used | Rules partition captures governance decisions in machine-readable form. Every governance cycle that updates Rules is a value locked in. |
| Dual architecture (deontic + probabilistic) | Rules = deontic skeleton. State = evidence layer. Core sits at the crystallization interface between them. |
| Beacons are the only API | Core services are beacons. No action bypasses them. The 2x2 matrix is structurally supported. |
| Single source of truth | The Synome is one logical system. Rules and State are partitions, not separate databases. |
| Evidence flows back | Data Ingest writes evidence into State continuously. Core reads it, applies Rules, writes derived state back. The loop is closed. |
| Default-deny | Access & Routing enforces. No permission = no access. Authority is granted explicitly, revocable at any time. |
| Append-only, auditable | State uses append-only writes. Block versioning handles reorgs. The audit trail is structural, not optional. |

---

## What's Deferred and Why

| Vision concept | Status | Why deferred |
|---|---|---|
| Teleonomes (Layer 3) | Not built | Private AI cognition requires the full five-layer stack. Phase 1 builds Layers 1-2 first. |
| Sentinels (HPHA) | Not built | Requires continuous real-time monitoring infrastructure. Arrives Phase 9-10. |
| High-Power beacons (HPLA/HPHA) | Not built | Bottom row of 2x2 matrix. Top row is sufficient for Phase 1 operations. |
| Probabilistic mesh with truth values | Simplified | State holds evidence but without (strength, confidence) pairs. Governance evaluates evidence manually. |
| Automated crystallization | Manual | Governance reviews evidence and updates Rules through human process. Automation comes later. |
| Language Intent | Not built as described | Translation from Atlas to machine rules is done by engineers writing specs. The single-translator pattern is aspirational. |
| Self-hosting (code in the data) | Partial | Rules are stored as data (config store). Loop templates and runtime source are not yet stored in the Synome. |
| Dreamer/actuator split | Not built | Requires teleonome infrastructure. Phase 9+. |
| RSI (recursive self-improvement) | Not built | Requires the probabilistic mesh, dreamer infrastructure, and crystallization automation. |

Every deferral is a complexity reduction, not a structural deviation. The architecture supports adding each of these without rewriting what exists. The foundations are laid. The invariants hold.

---

## The Path Forward

The architecture is designed to deepen, not to be replaced:

**Phase 1 (now):** Rules + State + Core + Data Ingest. The deontic skeleton and evidence layer are operational. Beacons execute. Evidence flows. Governance acts manually.

**Phases 2-4:** Settlement automation, daily cycles, LCTS launch. Core services expand. The crystallization interface gets more systematic.

**Phases 5-8:** Factories. Agent onboarding becomes repeatable. The Synome grows in scope but not in structural complexity.

**Phases 9-10:** Sentinels. The full 2x2 matrix is deployed. HPHA beacons provide continuous real-time control. Automated crystallization begins. The system starts to reason about itself.

**Beyond:** Teleonomes. The five-layer architecture realizes fully. Private cognition with regulated action. The governance window narrows but the values are locked in.

At every step, the architecture we have today remains load-bearing. Nothing gets thrown away. It deepens.

---

## Summary

The engineering architecture is not a simplification of the vision. It is the vision's Phase 1 skeleton — the deontic skeleton itself, built first because it must be load-bearing before anything else can hang off it. The probabilistic mesh, the teleonomes, the sentinels, the self-hosting — all of these are elaborations that build on top of what we have.

The Synome is real. It holds Rules and State. Core reads from it and writes back to it. Evidence flows in from the world. Governance crystallizes evidence into rules. Applications interact through regulated gates. The loop is closed.

Everything the vision requires is either built or structurally supported. Nothing is compromised.
