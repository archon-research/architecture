# Laniakea Architecture

Architecture documentation for Laniakea — the bridge between engineering implementation and Sky governance principles.

## Who This Is For

- **Engineering teams** — understand why components are designed the way they are, how they map to Synome concepts, and how the architecture evolves across phases
- **Business & governance teams** — see how Synome principles and Atlas constraints are operationalized in the system
- **Everyone** — one diagram, one vocabulary, one source of truth

## How It's Organized

| Folder | Contents |
|--------|----------|
| `decisions/` | Trade-offs considered and constraints assumed |
| `diagrams/` | Architecture diagrams (single diagram, readable by all audiences) |
| `mappings/` | Engineering ↔ Synome terminology mapping |
| `operational/` | Resilience patterns and scaling considerations |
| `reference/` | Glossary, technology stack, Synome concepts for engineers, governance model |
| `roadmap/` | Laniakea phase evolution and current Phase 1 status |

## Related Repositories

- [`stl`](https://github.com/archon-research/stl) — STL service family (Go). Currently houses `stl-verify` — the block watcher and indexer that forms the Phase-1 data spine.
- [`synome`](https://github.com/archon-research/synome) — Source-of-truth platform (specs, graph DB, Explorer UI)
- [`infrastructure`](https://github.com/archon-research/infrastructure) — Platform layer (clusters, GitOps, observability, networking)
- [`laniakea-docs`](https://github.com/archon-research/laniakea-docs) — Synome framework, Atlas, and business concepts

## Phase Context

The architecture evolves as Laniakea progresses through its phases — see `roadmap/laniakea-phase-evolution.md` for the full picture.
