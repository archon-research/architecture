# Synome — Rules

What categories of rules the Synome stores. This is the schema, not the values — specific parameters change at governance speed and live in the Synome itself.

The Synome has two partitions: **Rules** (this doc) and **[State](synome-state.md)**. Rules holds governance-set configuration; State holds operational data. Together they form the Synome — the canonical data layer that Core operates on.

For how the Synome relates to the rest of the architecture, see the [architecture diagram](../diagrams/laniakea-architecture.excalidraw).

---

## What Rules is

The slow-changing partition of the Synome. Machine-readable translations of human-readable governance decisions (the Atlas). Changes at governance speed — weeks, months. Answers **"what are the rules?"**

---

## Rule categories

| Category | What it contains | Examples |
|---|---|---|
| Risk formulas | How capital requirements are calculated | CRR formula, ER formula, ORC formula |
| Risk parameters | Values plugged into those formulas | CRR per asset type, covenant thresholds, stress scenario parameters |
| Settlement logic | How settlement is computed | Five-step calculation, auction mechanics, penalty formulas |
| Settlement parameters | Values for settlement | Cycle timing, penalty rates, spread targets |
| Agent definitions | What agent types exist and their relationships | Type hierarchy, rank rules, accordancy chain |
| Agent registry | Which specific agents exist | Spark is a Prime under Ozone at address 0x3300... |
| Authority chain | How permissions cascade | BEAM levels, freeze rules, threshold requirements |
| Book structure | Book types and their invariants | Assets=liabilities, book hierarchy, unit standards |
| Lifecycle rules | State machines and transition rules | Book states, attestation requirements |
| Token rules | Token definitions and mechanics | Peg rules, vault mechanics, emission rules |
| Matching rules | Duration matching and hedging logic | Pull-to-par rule, hedging requirements |
| Phase carve-outs | Deliberate temporary simplifications | What's deferred and why |

---

## How each category maps to the system

### Risk formulas
The math that Core services use to compute capital requirements. The formulas themselves rarely change. When they do, it's a major governance event.

### Risk parameters
The values plugged into those formulas. Change more often than formulas but still at governance speed — e.g., adjusting CRR for a new asset class, updating stress scenario parameters.

### Settlement logic
The rules for how settlement is computed: the five-step Prime settlement, auction mechanics (OSRC, duration bucket), penalty calculation formulas. Structural — changes are rare.

### Settlement parameters
The values used during settlement: cycle timing (active window, lock window), penalty rates, auction parameters, distribution spread targets. Tuned periodically by governance.

### Agent definitions
The structural definitions of agent types: what a Prime is vs a Halo vs a Guardian, rank hierarchy, governance relationships, transformation rules. Changes when new agent types are introduced.

### Agent registry
Which specific agents exist and their configuration: Spark is a Star Prime, accordant to Ozone, with SubProxy at 0x3300... Changes when agents are created, restructured, or retired.

### Authority chain
How permissions are structured: BEAM hierarchy (Council Beacon > aBEAM > cBEAM > pBEAM), freeze/cancel rules, threshold requirements (16/24 for spells), timelock rules. Changes when governance restructures authority.

### Book structure
The types of books that exist, their invariants (assets=liabilities), the hierarchy (Genbook > Primebook > Halobook), unit standards (LCTS, NFAT), and sub-book taxonomy (ascbook, tradingbook, termbook, structbook, hedgebook). Structural — changes are rare.

### Lifecycle rules
State machine definitions: book states (created > filling > ... > closed), valid transitions, what triggers each transition, attestation requirements. Changes when lifecycle mechanics are redesigned.

### Token rules
Token definitions: peg rules (USDS 1:1 USD), vault mechanics (sUSDS ERC-4626), convertibility rules (DAI/USDS 1:1), emission rules (Prime tokens permanently disabled), risk capital token types. Changes when new tokens are introduced or mechanics are altered.

### Matching rules
Duration matching and hedging logic: pull-to-par rules, what counts as matched vs unmatched, hedging requirements for fixed-rate exposure, tug-of-war allocation mechanics. Changes when the matching framework is redesigned.

### Phase carve-outs
Deliberate simplifications for the current phase: what's deferred, what's simplified, what temporary defaults are in place. These are explicitly temporary — they get removed as the system matures.
