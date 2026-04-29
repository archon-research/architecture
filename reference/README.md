# Reference Documents

Stable, slow-changing reference material that supports every other section of this repo. If a doc anywhere else cites a term, technology choice, or governance principle, the canonical definition lives here.

## Contents

| Document | What it covers | Read this when... |
|---|---|---|
| [glossary.md](glossary.md) | Definitions of business and engineering terms used across all five repos | You hit an unfamiliar term and need a one-line definition |
| [synome-concepts-for-engineers.md](synome-concepts-for-engineers.md) | Plain-English mapping of Atlas / Synome / Beacon / BEAM / Sentinel into engineering vocabulary | You're reading `laniakea-docs` for the first time, or a design doc that uses Synome terminology |
| [security-governance-model.md](security-governance-model.md) | How protocol governance and platform governance are enforced, and where they meet | You're designing a new beacon, reviewing access changes, or planning a Sentinel-related feature |
| [technology-stack-matrix.md](technology-stack-matrix.md) | High-level technology choices per layer, with rationale | You need to know *what* is used (or *why* it was chosen) without leaking deployment specifics |

## Audience and level

These documents are deliberately **conceptual and high-level**. They name technology categories (managed Kubernetes, time-series database, FIFO message stream) but not specific deployments, accounts, regions, IP ranges, or vendor product names tied to Sky's environments. Operational specifics belong in the relevant engineering repo's runbooks.

## How to use

- **New engineers**: read `synome-concepts-for-engineers.md` first, then `glossary.md` as a lookup. The other two are reference material to consult when needed.
- **Designing a new component**: cross-check it against `security-governance-model.md` (does it fit the authority model?) and the relevant engineering-side mapping in `mappings/`.
- **Discussing strategy**: `glossary.md` is the shared vocabulary — link to it in design docs rather than re-defining terms inline.

## Cross-references

- `mappings/` — translation tables between business and engineering vocabularies for specific architectures
- `decisions/` — significant architecture decisions, the trade-offs behind them, and the constraints they assume
- `roadmap/` — how the architecture evolves through Laniakea phases
- `operational/` — resilience, scaling, and runtime concerns
