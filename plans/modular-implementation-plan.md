# Imperio Inmobiliario: Modular Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Decompose the ~6400-line monolithic spec (v12.3) into independently testable modules with microservice-like boundaries.

**Architecture:** Modular monolith — 14 npm packages under `packages/` with explicit interfaces and dependency injection. Each module testable in isolation without full game running.

**Tech Stack:** TypeScript strict mode, Colyseus 0.15+, Node.js 20+, PostgreSQL 15+, Vitest for testing.

---

## Module Overview (14 Packages)

```
packages/
├── schema/          # L0: Types, interfaces, Colyseus schemas
├── data/            # L1: Satellite JSON loader + validation
├── state-machine/   # L2: 11-phase game loop orchestration
├── economy/         # L2: Wealth, Rent, OpsCost, LeaderTax formulas
├── property/        # L2: Ownership, upgrades, packaging
├── auction/         # L2: Sealed bidding, tie-breaks
├── action/          # L3: Major/Minor action processing
├── deal/            # L3: 7 deal templates, price validation
├── event/           # L2: RiskScore, targeting, fairness caps
├── election/        # L2: Mood/election voting, meters
├── bot/             # L3: AI heuristics H1-H9
├── persistence/     # L3: Graf Dollar ledger, snapshots
├── press/           # L2: Spanish headline generation
└── room/            # L4: Colyseus ImperioRoom orchestrator
```

---

## Dependency Graph

```
L0: schema (no deps)
     ↓
L1: data → schema
     ↓
L2: economy, property, auction, event, election, press, state-machine → schema, data
     ↓
L3: action → property, economy
    deal → property, economy
    bot → auction, action, deal
    persistence → schema
     ↓
L4: room → ALL modules
```

**Rule:** Modules can only depend on same or lower levels. No circular deps.

---

## Implementation Phases

### Phase 1: Foundation (Week 1-2)

#### Task 1.1: Initialize Monorepo
- Create `packages/` structure with npm workspaces
- Configure TypeScript project references
- Set up Vitest for testing
- Create shared tsconfig.base.json

#### Task 1.2: @imperio/schema
**Files:** `packages/schema/src/`
- `types.ts` — All type aliases (Phase, InsuranceLevel, DealTemplateId, etc.)
- `interfaces.ts` — All interfaces from spec A1 (OwnerState, PropertyState, etc.)
- `schemas.ts` — Colyseus @type decorated classes
- `index.ts` — Re-exports

**Test:** Type compilation only (no runtime tests needed)

#### Task 1.3: @imperio/data
**Files:** `packages/data/src/`
- `loader.ts` — JSON file loading with validation
- `validators.ts` — Zod schemas for each JSON file
- `accessors.ts` — Type-safe getters (getProperty, getEvent, etc.)

**Test:** Load actual JSON files, validate schema compliance

#### Task 1.4: @imperio/state-machine
**Files:** `packages/state-machine/src/`
- `phases.ts` — Phase enum and ordering
- `machine.ts` — GameStateMachine class
- `timers.ts` — Phase duration configs by mode

**Test:** Phase transitions, timeout handling, message gating

---

### Phase 2: Core Economy (Week 3-4)

#### Task 2.1: @imperio/economy
**Files:** `packages/economy/src/`
- `wealth.ts` — `calculateWealth()`, `calculatePropertyWorth()`
- `rent.ts` — `calculateRent()`, `calculateDistrictSetBonus()`, `calculateSignalBonus()`
- `costs.ts` — `calculateOpsCost()`, `calculateLeaderTax()`
- `income.ts` — `processIncome()` orchestrator
- `forced-sales.ts` — `processForcedSales()`

**Key Formulas (spec Part 6):**
```typescript
Wealth = Cash + Σ(PropertyWorth) - Debt
PropertyWorth = BasePrice + (UpgradeLevel × 40_000) + WorthBonuses
Rent = BaseRent + (UpgradeLevel × 12_000) + DistrictBonus + SignalBonus + PackageBonus
OpsCost = floor((NumProperties + TotalUpgrades) / 3) × 12_000
LeaderTax = 5% of TotalRent (top quartile) | 10% (if #1 > 1.2× #2)
```

**Test:** Property-based tests for formula invariants, edge cases (0 properties, max debt)

#### Task 2.2: @imperio/property
**Files:** `packages/property/src/`
- `ownership.ts` — Transfer, getOwnerProperties
- `upgrades.ts` — canUpgrade, upgrade, getUpgradeCost
- `insurance.ts` — canInsure, insure
- `packaging.ts` — canCreatePackage, createPackage, dissolvePackage
- `districts.ts` — getDistrictSets, hasDistrictSet

**Test:** Package creation requirements, dissolution rules, upgrade level gates

---

### Phase 3: Game Actions (Week 5-6)

#### Task 3.1: @imperio/auction
**Files:** `packages/auction/src/`
- `bids.ts` — Bid collection and validation
- `resolution.ts` — resolveAuction, resolveAllAuctions
- `tiebreak.ts` — Deterministic tie-break (amount → fame → coin flip)

**Tie-break (spec 5.5.2):**
```typescript
function tieBreak(roundSeed: string, propertyId: string, ownerIds: string[]): string {
  const sorted = ownerIds.slice().sort();
  const hash = sha256(`${roundSeed}|${propertyId}|${sorted.join(',')}`);
  return sorted[parseInt(hash.slice(0, 8), 16) % sorted.length];
}
```

**Test:** Tie-break determinism, fame token priority, bid validation

#### Task 3.2: @imperio/action
**Files:** `packages/action/src/`
- `validation.ts` — canSubmitAction, validateMajorAction, validateMinorAction
- `major.ts` — processDevelop, processPackage, processDeal, processPass
- `minor.ts` — processInsure, processSecurity, processPRPush, processComplianceAudit
- `fame-spend.ts` — Yape, Control de daños, Cortina de humo

**Test:** Action slot constraints, cost validation, effect application

#### Task 3.3: @imperio/deal
**Files:** `packages/deal/src/`
- `templates.ts` — 7 deal template definitions
- `validation.ts` — Price range validation (0.6x-2.0x)
- `lifecycle.ts` — createDeal, respondToDeal
- `resolution.ts` — resolveDeals (sorted by dealId)
- `bank-loan.ts` — Loan creation, repayment scheduling
- `suggestions.ts` — Smart deal suggestions

**Price Validation (spec 7.2.2b):**
```typescript
CASH_FOR_PROPERTY: amount >= BasePrice × 0.6 && amount <= BasePrice × 2.0
SWAP: abs(Price_A - Price_B) <= max(Price_A, Price_B) × 0.5
SELL_PACKAGE: amount >= sum(BasePrices) × 0.6
```

**Test:** Template validation, conflict resolution, loan repayment

---

### Phase 4: Events and Voting (Week 7-8)

#### Task 4.1: @imperio/event
**Files:** `packages/event/src/`
- `risk-score.ts` — calculateRiskScore with all modifiers
- `targeting.ts` — selectTargets weighted by RiskScore
- `selection.ts` — selectEvents by category weights
- `resolution.ts` — resolveEvent with mitigations
- `fairness.ts` — applyFairnessCaps
- `choice.ts` — createChoice, processChoiceTimeout (AutoChoice)
- `infrastructure.ts` — activateInfra (E-053 enables expropriation)

**RiskScore Formula (spec 8.1):**
```typescript
RiskScore = 1.0
  + 0.35 × NumProperties
  + 0.20 × FameTier
  + 0.20 × WealthTier
  + 0.25 × PoliticalExposure
  - 0.25 × Compliance
  + modifiers  // clamped [0.5, 3.5]
```

**Fairness Caps (spec 8.3):**
- Max GD 150K cash damage/round
- Max 1 expropriation/match
- Max 1 event ≥100K per 2-round window

**Test:** RiskScore clamping, fairness cap enforcement, AutoChoice logic

#### Task 4.2: @imperio/election
**Files:** `packages/election/src/`
- `mood.ts` — selectMoodOptions, submitMoodVote, resolveMood
- `election.ts` — selectCandidates, submitElectionVote, resolveElection
- `meters.ts` — getMeter, setMeter, adjustMeter
- `effects.ts` — applyMoodEffect, applyElectionBundle

**Test:** Watcher vs Owner voting fallback, tie-breaks, meter bounds

---

### Phase 5: AI and Persistence (Week 9-10)

#### Task 5.1: @imperio/bot
**Files:** `packages/bot/src/`
- `heuristics.ts` — H1-H9 decision functions
- `bidding.ts` — decideBid (H1-H2)
- `actions.ts` — selectAction (H5)
- `voting.ts` — decideMoodVote, decideElectionVote (H6)
- `packaging.ts` — decidePackage (H7)
- `fame.ts` — decideFameSpend (H8)
- `deals.ts` — evaluateDeal (H9)

**Bot Reserve:** GD 100,000 standardized (spec H-RESERVE)

**Test:** Heuristic consistency, reserve enforcement, difficulty scaling

#### Task 5.2: @imperio/persistence
**Files:** `packages/persistence/src/`
- `snapshots.ts` — saveSnapshot, loadSnapshot, getMatchHistory
- `ledger.ts` — award, getBalance, getLedger (double-entry)
- `achievements.ts` — unlockAchievement, getAchievements
- `leaderboard.ts` — getLeaderboard, getDynastyLeaderboard
- `decay.ts` — applySeasonalDecay

**Graf Dollar Ledger:** Double-entry accounting per `graf_dollar.txt`

**Test:** Idempotency key deduplication, balance consistency, decay calculation

#### Task 5.3: @imperio/press
**Files:** `packages/press/src/`
- `personas.ts` — 5 press personas with styles
- `headlines.ts` — generateHeadlines, interpolateTemplate
- `validation.ts` — 70-char headline limit

**Test:** Template interpolation, character limits

---

### Phase 6: Integration (Week 11-12)

#### Task 6.1: @imperio/room
**Files:** `packages/room/src/`
- `ImperioRoom.ts` — Colyseus Room orchestrating all modules
- `handlers/` — Message handlers (bid, action, deal, vote, etc.)
- `lifecycle.ts` — onCreate, onJoin, onLeave
- `reconnection.ts` — 60s reconnect window, bot activation

**Test:** Full round cycle e2e, reconnection flow, phase gating

#### Task 6.2: Integration Tests
- Multi-module interaction tests
- Full match simulation (4 players, 7 rounds)
- Concurrent bid resolution
- Event cascade validation

#### Task 6.3: Load Testing
- 12 concurrent players
- 20 participants (Owners + Watchers)
- Message throughput under load

---

## Testing Strategy

### Per-Module Testing

| Module | Strategy | Coverage Target |
|--------|----------|-----------------|
| schema | Type compilation | N/A |
| data | JSON validation | 100% |
| state-machine | State transitions | 90% |
| economy | Property-based (fast-check) | 95% |
| property | Unit + edge cases | 90% |
| auction | Deterministic tie-break | 95% |
| action | Validation rules | 90% |
| deal | Template validation | 95% |
| event | Fairness caps | 95% |
| election | Voting logic | 90% |
| bot | Heuristic consistency | 80% |
| persistence | Idempotency | 90% |
| press | Template limits | 85% |
| room | E2E integration | 80% |

### Test Commands
```bash
# Single module
npm test -w @imperio/economy

# All modules
npm test

# Coverage
npm test -- --coverage

# E2E only
npm test -w @imperio/room -- --testPathPattern=e2e
```

---

## Critical Files Reference

| File | Purpose |
|------|---------|
| `imperio_master_spec_v11.md` | Master spec (v12.3), single source of truth |
| `graf_dollar.txt` | Ledger schema and accounting rules |
| `data/properties.json` | 25-property deck |
| `data/events.json` | 52+ event deck |
| `data/moods.json` | 8 mood options |
| `data/signals.json` | 10 market signals |
| `data/teams.json` | 17 PARODY + 17 REAL dynasties |

---

## Verification

1. **Type Safety:** `tsc --noEmit` passes for all packages
2. **Unit Tests:** `npm test` all green, coverage thresholds met
3. **Integration:** Full 7-round match completes without errors
4. **Load Test:** 12 players, <100ms message latency at p99
5. **Spec Compliance:** Manual audit against spec formulas

---

## Execution Options

After approval:
1. **Subagent-Driven (this session)** — Fresh subagent per task, review between tasks
2. **Parallel Session (separate)** — Open new session with executing-plans skill
