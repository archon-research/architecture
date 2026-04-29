# Laniakea Roadmap

How Laniakea's architecture evolves across the eleven phases (0–10), and where Phase 1 stands today.

## Contents

| Document | What it covers | Read this when... |
|---|---|---|
| [laniakea-phase-evolution.md](laniakea-phase-evolution.md) | The architectural arc from Foundation (0–4) → Factories (5–8) → Automation (9–10) | You need to know what changes in later phases, or whether a design is reaching beyond Phase 1 |
| [phase-1-status.md](phase-1-status.md) | High-level status of Phase 1: built, in progress, explicitly deferred | You're scoping new work and need to know whether something already exists |

## How the roadmap relates to the rest of this repo

- **`reference/`** answers *what* and *why* (terms, technology choices, governance model).
- **`mappings/`** answers *how this vocabulary maps to that one* for a specific architecture.
- **`decisions/`** records significant architectural choices and their trade-offs.
- **`operational/`** describes how the running system behaves under stress.
- **`roadmap/`** (this folder) answers *when in the lifecycle* a capability lands.

## Cadence

This is a living view, not a delivery schedule. The phase grouping (Foundation / Factories / Automation) is durable; what moves is the boundary between "in progress" and "complete" inside each phase. Source of truth for ticket-level status remains the engineering repos.

## Cross-references

- `laniakea-docs/roadmap/` — the canonical phase-by-phase business roadmap
- `reference/synome-concepts-for-engineers.md` — the vocabulary used here
- `mappings/synome-engineering-business-mapping.md` — Phase 1 architecture in both vocabularies
