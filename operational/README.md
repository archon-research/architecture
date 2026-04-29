# Operational Considerations

How the architecture behaves at runtime: under partial failure, under load, and as it grows. These documents are conceptual — they describe patterns and behaviours, not specific incidents, hostnames, or capacity numbers.

## Contents

| Document | What it covers | Read this when... |
|---|---|---|
| [resilience-patterns.md](resilience-patterns.md) | Per-component failure modes and the patterns that make them recoverable | You're designing a new component, reviewing an outage, or evaluating a proposed change for blast radius |
| [scaling-considerations.md](scaling-considerations.md) | How each layer scales, where the natural ceilings are, and which growth vectors load which components | You're capacity-planning, onboarding a new source protocol, or evaluating whether a design generalises |

## How to read these docs

Both documents are organised by architectural layer (data spine → beacons → source of truth → platform), with cross-cutting patterns called out separately. They are intentionally short on specifics; for runbooks, capacity numbers, and current incident responses, consult the engineering repos.

## Cross-references

- `reference/technology-stack-matrix.md` — the technology choices that determine the operational characteristics
- `reference/security-governance-model.md` — the authority and access patterns that bound operational risk
- `roadmap/laniakea-phase-evolution.md` — how operational concerns change as the system gains Sentinels and factories
