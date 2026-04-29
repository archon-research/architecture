# Architecture Decisions

The significant architectural choices made for Laniakea Phase 1 — what we picked, what we considered, why, and what the cost is. These docs are conceptual; specific products, regions, accounts, and instance details are intentionally not named.

## Contents

| Document | What it covers | Read this when... |
|---|---|---|
| [constraints-and-assumptions.md](constraints-and-assumptions.md) | The hard and soft constraints, plus the assumptions about the world, that bound Phase 1 | You're reviewing or proposing a design and need to know which boundaries cannot move |
| [trade-offs.md](trade-offs.md) | The major decisions made inside those constraints, with alternatives, principles, and costs | You're about to argue for a different choice, or onboarding and trying to understand "why this and not that" |

## How to use this section

- **Reviewing a design.** First check `constraints-and-assumptions.md`. A design that violates a *hard* constraint needs re-architecture, not a workaround. A design that crosses a *soft* constraint or *assumption* is a candidate for a new ADR.
- **Onboarding.** Read these two documents in order. Together they explain why the system looks the way it does without requiring you to dig through every engineering repo.
- **Adding a new decision.** Append to `trade-offs.md` using the entry template at the bottom of that file. Decisions should name realistic alternatives, the principle that decided them, the cost paid, and the signal that would change the answer.

## What is *not* here

- **Per-component design notes** — those live in the engineering repo for that component.
- **Runbooks and incident retros** — those live in the engineering repo's operational docs.
- **Spec change history for the Atlas** — that lives in the Synome graph database's branch history.

## Cross-references

- `reference/security-governance-model.md` — protocol-side decisions about authority, access, and enforcement
- `reference/technology-stack-matrix.md` — the per-layer technology picks driven by these decisions
- `operational/resilience-patterns.md` and `operational/scaling-considerations.md` — how the decisions behave at runtime
- `roadmap/laniakea-phase-evolution.md` — when soft constraints and assumptions change shape
