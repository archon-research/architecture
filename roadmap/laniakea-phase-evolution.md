# Laniakea Phase Evolution

How the architecture changes across Laniakea's eleven phases (0–10). This is a high-level architectural view, not a delivery schedule.

The phases group into three eras:

- **Foundation (0–4)** — establish the institutional layer, deterministic execution, the data spine, and the daily settlement cadence.
- **Factories (5–8)** — turn one-off agents into reproducible patterns. Onboarding new Primes, Halos, and Generators becomes a configuration change.
- **Automation (9–10)** — Sentinels arrive, the probabilistic mesh activates, and the system shifts from "humans approving everything" to "humans approving exceptions."

Every phase preserves the architectural pattern of the previous one. The model accumulates capabilities; it does not replace them.

---

## The big picture

```
Foundation                 Factories                   Automation
(Phase 0–4)                (Phase 5–8)                 (Phase 9–10)
  │                          │                           │
  │  Deterministic           │  Reproducible              │  Self-regulating
  │  beacons                 │  agent factories           │  via Sentinels
  │                          │                           │
  ▼                          ▼                           ▼
LPLA + LPHA only       + Halo / Prime /              + HPHA Sentinels
Manual onboarding        Generator factories         + Probabilistic mesh
Human-in-the-loop      ~80% automation               99% automation
~40–50% automation
```

The shift is not about adding more services. It is about changing what humans do: from manually approving every action, to approving every *new pattern*, to approving only exceptions.

---

## Foundation (Phases 0–4)

**Architectural goal:** stand up the source-of-truth platform, the on-chain authority hierarchy, the data spine, and the daily settlement cadence. Make every protocol action explicitly governed and observable.

| Phase | Architectural addition | Why it matters |
|---|---|---|
| **0** | Sky Ecosystem reorganised. Atlas v0 written. | Establishes the constitution that everything else encodes. |
| **1** | Synome-MVP. Beacon framework deployed (LPLA + LPHA). aBEAM/cBEAM/pBEAM hierarchy live. PAU pattern (Diamond) for Primes. STL block-data service indexes everything. | The institutional layer becomes real. Every action now happens through a registered, scoped, anchored, observable, revocable beacon. |
| **2** | LCTS launches. Generator issues `srUSDS` via the queue-based conversion system. NFAT (bespoke Term deals) live. | Capacity-constrained capital movement is now first-class. Bespoke deals get a real token primitive. |
| **3** | Daily Settlement Cycle activates. Active Window 16:00→13:00 UTC; Lock 13:00→16:00; Moment of Settlement at 16:00. | The protocol gets a heartbeat. Risk capital, allocations, and reservations now reconcile on a known cadence. |
| **4** | Configurator Unit fully replaces spell-based governance for Prime operations. ALDM, Lindy Duration Model, SPTP active. | Day-to-day Prime config no longer needs a spell. Asset-liability matching is enforced by code. |

What the architecture looks like at the end of Foundation:

- Every Synomic Agent has a PAU.
- Every action a Synomic Agent takes goes through a beacon.
- Every beacon has an explicit authority envelope and is registered in Synome.
- The Council Beacon (HPHA) is the only path to grant or revoke high-authority operations.
- The data spine reconstructs full state at any block, including across reorgs.
- Humans approve every meaningful change to Atlas constraints, agent registry, and authority grants.

What Foundation deliberately *does not* include: HPHA Sentinels, factory patterns for new agents, automated crystallization, auction-based OSRC pricing, Trading Halos, Identity Networks.

---

## Factories (Phases 5–8)

**Architectural goal:** turn each agent class into a *factory*. Onboarding the n-th Prime, the n-th Halo, or the n-th Generator should be a configuration change, not a bespoke design exercise.

| Phase | Architectural addition |
|---|---|
| **5** | **Prime Factory.** Standard PAU template, standard Configurator Unit, standard beacon roster. Onboarding a new Prime is a series of registrations, not a redesign. |
| **6** | **Halo Factory.** Portfolio Halo (LCTS-backed) and Term Halo (NFAT-backed) become standard products. Trading Halo (AMM-backed) introduced. Buybox parameters become first-class config. |
| **7** | **Generator Factory.** Multiple Generators can coexist. Each manages its own interface to the stablecoin contract; each issues its own `srUSDS`-class token. |
| **8** | **Risk-capital factories.** TEJRC and TISRC become standard primitives. Auctions for OSRC become operational. The encumbrance ratio is enforced by code. |

What changes architecturally between Foundation and the end of Factories:

- The set of *kinds* of agents is fixed. The set of *instances* grows freely.
- The Synome registers more, the application code changes less.
- Beacon classes (LPLA / LPHA / HPLA) all see use; HPHA still deferred.
- Code paths previously unique to a single Prime become shared library code.

The principle: **Phase 1 made the system governable. Phases 5–8 make it scalable without re-governing each instance.**

---

## Automation (Phases 9–10)

**Architectural goal:** Sentinels watch, react, and bound damage. Crystallization of probabilistic evidence into deontic axioms moves from manual review toward automation. Human approval shifts from "every action" to "exceptions and Atlas changes."

| Phase | Architectural addition |
|---|---|
| **9** | **Sentinels active.** Four classes: Baseline (`stl-base`), Stream (`stl-stream`), Warden (`stl-warden`), Principal (`stl-principal`). HPHA authority granted. Auctions price OSRC. The `Rate Limit × TTS` damage bound becomes operational. |
| **10** | **Probabilistic mesh.** Sentinels coordinate. Beacons feed Sentinels with attestations. Crystallization automation: well-attested observations promote into binding spec on a defined review cadence. |

What changes architecturally between Factories and Automation:

- HPHA appears for the first time. The 2×2 matrix is fully populated.
- The `lpha-relay` is no longer the last line of defence — Sentinels are.
- The Operational Risk Capital charge is real, sized, and enforced.
- The Synome stops being a passive store and starts participating in feedback loops (probabilistic mesh).
- Human attention shifts from approving *what beacons do* to approving *what becomes Atlas*.

Notably, **the structures and authority model from Phase 1 do not change.** A Phase-9 architecture is recognisable as a Phase-1 architecture with Sentinels added. This is intentional — the BEAM hierarchy and beacon contract were designed with Sentinels in mind from the start.

---

## What carries through every phase

Some properties are invariant across all eleven phases. If a future change appears to violate one of these, treat it as a red flag worth escalating:

1. **Every action goes through a beacon.** No back doors, no off-band administrative paths.
2. **Every beacon is registered, scoped, anchored, observable, revocable.** The five non-negotiables.
3. **Authority flows through aBEAM → cBEAM → pBEAM.** The high-authority components are always the lowest-frequency.
4. **All authority grants are revocable; high-authority additions are timelocked.**
5. **Atlas changes go through human review.** Crystallization automation in Phase 10 narrows the human path; it does not eliminate it.
6. **The data spine is append-only and reconstructable.** Block versioning carries through every phase.

---

## Cross-references

- `roadmap/phase-1-status.md` — what is built, in progress, and remaining inside Foundation
- `reference/synome-concepts-for-engineers.md` — the vocabulary used above (Beacon, BEAM, Sentinel, etc.)
- `reference/security-governance-model.md` — how the BEAM hierarchy and beacon contract enforce these invariants
- `mappings/synome-engineering-business-mapping.md` — Phase 1 components mapped to Synome roles
