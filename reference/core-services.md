# Core Services Schema

What the Core layer does. Core is the compute layer that reads from the Synome (both Rules and State), processes, and writes results back to State.

For how Core relates to the rest of the architecture, see the [architecture diagram](../diagrams/laniakea-architecture.excalidraw).

---

## What Core is

Core is the compute layer — the services that operate on the Synome. Each service reads rules and state, runs its logic, and writes derived state back.

```
Synome Rules + Synome State → Core services → Derived state → Synome State
```

---

## Service categories

| Service | What it does | Reads from | Writes to |
|---|---|---|---|
| Risk calculation | Computes CRR, ER, unit risk weights per agent | Rules: risk formulas, parameters. State: positions, prices | State: risk scores, breach records |
| Oracle service | Ingests and validates oracle price feeds | State: raw price data | State: validated oracle prices |
| Settlement | Runs per-Prime and global settlement at each epoch boundary | Rules: settlement logic, fee schedules. State: positions, risk scores, breach records | State: settlement records, penalty calculations |
| NFAT lifecycle | Manages NFAT state transitions (filling → deploying → at-rest → ...) | Rules: lifecycle rules, transition rules. State: NFAT records, attestations | State: updated NFAT records |
| Attest workflow | Processes attestation submissions and validates them against rules | Rules: attestation requirements, permission rules. State: current NFAT state | State: attestation records |
| Breach monitoring | Watches for covenant violations in real time | Rules: covenant thresholds. State: current ER per agent | State: breach records |
| Auction engine | Runs OSRC and duration bucket auctions at settlement | Rules: auction mechanics, parameters. State: bids, capacity | State: auction results, allocations |
| Distribution | Calculates reward distributions per agent | Rules: distribution formulas, fee schedules. State: settlement records | State: distribution records |

---

## How each service works

### Risk calculation
The central computation. Takes positions and prices from State, applies CRR formulas and risk parameters from Rules, produces per-position risk weights and per-agent encumbrance ratios. Runs continuously or on price changes.

### Oracle service
Validates and normalizes price data from chain oracles. May cross-reference multiple sources, flag stale prices, or apply confidence weights. Output is the canonical price set that risk calculation and settlement use.

### Settlement
The periodic batch job. At each epoch boundary: runs the five-step per-Prime settlement in parallel, waits for on-chain confirmations, then runs global aggregation. Produces the definitive financial records for each cycle.

### NFAT lifecycle
Manages individual NFAT state transitions. Enforces the state machine defined in Rules — e.g., a book can't move to "deploying" without an attestation. Reads current NFAT state from State, validates transition requests against Rules, writes updated state back.

### Attest workflow
Handles attestation submissions from authorized beacons. Checks that the attester has permission (Rules auth), that the attestation target exists and is in the right state (State), and records the attestation event. Triggers downstream lifecycle transitions when attestation requirements are met.

### Breach monitoring
Continuously compares current encumbrance ratios (from risk calculation) against covenant thresholds (from Rules). When a breach is detected, creates a breach record with timestamp and severity. Breach records feed into penalty calculations at settlement.

### Auction engine
Runs sealed-bid uniform-price auctions for OSRC capacity and duration buckets. Reads auction parameters from Rules, collects bids, executes matching algorithm, writes results. OSRC runs daily with no multi-day reservations; duration bucket auctions allow multi-epoch reservations.

### Distribution
Calculates what each agent earns or owes beyond the base settlement. Applies distribution reward formulas, integration boosts, ecosystem upkeep rules from Rules to the settlement outputs in State.
