# Imperio Inmobiliario: Modular Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Decompose the ~6200-line monolithic spec (v12.4) into independently testable modules with microservice-like boundaries. Includes both server (TypeScript/Colyseus) and client (Godot 4.5+) implementation.

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

## CLIENT IMPLEMENTATION (Phases 7-12)

> **Foundation Commitments (Spec v12.4, Appendix O):**
> - Card UI: `chun92/card-framework` (MIT, Godot 4.5+)
> - WebSocket patterns: `alpapaydin/Godot4-Multiplayer` (MIT)
> - See spec Section 11.2 for full architecture

---

### Phase 7: Godot Client Foundation (Week 13-14)

#### Task 7.1: Initialize Godot Project
- Create Godot 4.5+ project with GL Compatibility renderer
- Configure project settings for WebGL2 export
- Set up directory structure per spec 11.2.2:
```
client/
├── project.godot
├── addons/card-framework/
├── autoloads/
├── scenes/
├── data/
└── assets/
```

#### Task 7.2: Install Card Framework
**Source:** `chun92/card-framework` (copy to `addons/card-framework/`)
- Enable plugin in Project Settings
- Verify Card, Pile, Hand, CardManager work
- Test basic drag/drop functionality

**Test:** Instantiate Card, add to Pile, move to Hand

#### Task 7.3: Create Autoloads
**Files:** `client/autoloads/`
- `Constants.gd` — Server URL, port, SSL toggle, cert path
- `ImperioClient.gd` — WebSocket client (spec 11.2.4)
- `GameState.gd` — Local state mirror, signal hub

**Key Signals:**
```gdscript
signal connected
signal disconnected
signal state_changed(state: Dictionary)
signal phase_changed(phase: String, ends_at: int)
signal error_received(code: String, message: String)
```

**Test:** Connect to local Colyseus server, receive state snapshot

#### Task 7.4: WebSocket SSL Support
Adapt from `alpapaydin/Godot4-Multiplayer` patterns:
- `USE_SSL` toggle in Constants
- Certificate loading for native builds
- Browser handles SSL for web exports

**Test:** Connect via `wss://` to HTTPS server

---

### Phase 8: Card UI Integration (Week 15-16)

#### Task 8.1: PropertyCard Extension
**Files:** `client/scenes/match/components/PropertyCard.gd`
```gdscript
class_name PropertyCard
extends Card

var property_id: String
var owner_id: String
var upgrade_level: int = 0
var is_insured: bool = false
var package_id: String = ""

func update_from_state(property_state: Dictionary):
    # Sync from Colyseus state
```

**Visual Elements:**
- GD price label
- District badge
- Upgrade stars (1-5)
- Insurance shield icon
- Package indicator

**Test:** Create card from JSON, update from state dict

#### Task 8.2: MarketRow Container
**Files:** `client/scenes/match/components/MarketRow.gd`
- Extends `Pile` from card-framework
- Horizontal layout for 5-6 cards
- Emits `property_selected(property_id)` on click
- Highlight affordability based on player cash

**Test:** Add 5 cards, select one, verify signal

#### Task 8.3: Portfolio Container
**Files:** `client/scenes/match/components/Portfolio.gd`
- Extends `Hand` from card-framework
- Shows player's owned properties
- Groups by district (visual indicator)
- Package grouping with border

**Test:** Add cards, verify layout, check district grouping

#### Task 8.4: State Synchronization
**Files:** `client/scenes/match/match.gd`
- Connect to `ImperioClient.state_changed`
- Create/update PropertyCards from state
- Move cards between containers on ownership change
- Handle property reveal animation

**Sync Flow:**
1. `STATE_SNAPSHOT` → Full rebuild
2. `OP_DELTA` → Incremental updates
3. `PHASE_CHANGE` → UI transitions

**Test:** Full round cycle with 4 players, cards move correctly

---

### Phase 9: Match UI (Week 17-18)

#### Task 9.1: Phase Panels
**Files:** `client/scenes/match/phases/`

| Panel | Elements | User Actions |
|-------|----------|--------------|
| `BidPanel.tscn` | Property cards, bid input, timer | Select property, enter bid amount |
| `ActionPanel.tscn` | Major/Minor slots, action buttons | Select action, confirm |
| `DealPanel.tscn` | Template dropdown, partner select, price input | Create offer, respond to offers |
| `MoodPanel.tscn` | 3 mood options, vote buttons | Cast vote |
| `ElectionPanel.tscn` | 4 candidates, vote buttons | Cast vote |

**Phase Timer:** Countdown bar with urgency colors (green → yellow → red)

#### Task 9.2: Visual Ledger
**Files:** `client/scenes/match/components/VisualLedger.gd`
- Animated income breakdown (spec V4.1)
- Staggered item appearance (0.2s each)
- Running cash counter
- Adaptive timing (3.0s base + 0.15s/item, max 5.0s)

**Animation Sequence:**
1. Fade in panel
2. Income items (green, staggered)
3. Cost items (red, staggered)
4. Total with scale punch
5. Cash counter animation
6. Fade out

**Test:** 8-item ledger completes in <5s

#### Task 9.3: Quick Chat
**Files:** `client/scenes/match/components/QuickChat.gd`
- 12 preset buttons (loaded from `quickchat.json`)
- 2 messages/phase rate limit (client-side hint)
- Floating message display over sender

**Test:** Send message, verify rate limiting

#### Task 9.4: Event Overlay
**Files:** `client/scenes/match/components/EventOverlay.gd`
- Full-screen overlay for events
- Event name, description (Spanish)
- Target player highlight
- 10s choice window for CHOICE events
- Insurance mitigation display

**Test:** Display event, verify timer, handle choice

#### Task 9.5: HUD Elements
**Files:** `client/scenes/match/hud/`
- `CashDisplay.tscn` — Current GD with animated changes
- `PhaseTimer.tscn` — Countdown with phase name
- `PlayerList.tscn` — 12 players, wealth rank, connection status
- `RoundIndicator.tscn` — Current round / max rounds

---

### Phase 10: Polish & Export (Week 19-20)

#### Task 10.1: Mobile Input
- Bid amounts: Preset buttons (+50K, +100K, +500K, MAX)
- Property selection: Tap on card (no drag required)
- Deal prices: Numeric input with validation
- HTML DOM overlay for text inputs (profile name, chat)

**Touch Targets:** Minimum 44x44 pixels

#### Task 10.2: Audio Integration
**Files:** `client/assets/audio/`, `client/scenes/match/Audio/`
- Ambient layers (spec V6.1):
  - `lobby_ambient.ogg`
  - `bid_tension.ogg`
  - `action_calm.ogg`
  - `event_drama.ogg`
- SFX: bid_placed, auction_won, card_move, event_hit
- Volume ducking for events

#### Task 10.3: Responsive Layout
- Test on 16:9 (desktop), 9:16 (mobile portrait), 4:3 (tablet)
- Card scaling based on viewport
- HUD repositioning for mobile

#### Task 10.4: Web Export
**Files:** `export_presets.cfg`
```ini
[preset.0]
name="Web"
platform="Web"
vram_texture_compression/for_desktop=true
vram_texture_compression/for_mobile=true
html/canvas_resize_policy=2
html/experimental_virtual_keyboard=false
```

**Test:** Export, deploy to test server, verify on Chrome/Firefox/Safari

#### Task 10.5: SSL Certificate Deployment
- Generate Let's Encrypt certs
- Configure Colyseus server with HTTPS
- Update `Constants.USE_SSL = true`
- Verify `wss://` connection from web client

---

### Phase 11: Client-Server Integration (Week 21-22)

#### Task 11.1: Full Round Cycle
- 4 human-controlled clients + 8 bots
- Complete 7-round Quick match
- Verify all phases work end-to-end

**Checklist:**
- [ ] REVEAL shows correct properties
- [ ] BID resolves correctly, cards move
- [ ] INCOME_CALC triggers Visual Ledger
- [ ] ACTION submits Major/Minor
- [ ] MOOD/VOTE voting works
- [ ] DEALS create and resolve
- [ ] EVENTS display and resolve
- [ ] PRESS headlines appear
- [ ] Final rankings correct

#### Task 11.2: Reconnection Flow
- Player disconnects mid-match
- 60s reconnection window
- Bot takes over if timeout
- Reconnect restores full state

#### Task 11.3: Watcher Mode
- Join as Watcher (13th+ participant)
- Read-only state sync
- Quick Chat enabled (viewer participation)
- No action/bid/vote capabilities

#### Task 11.4: Error Handling
- Server sends `ERROR` message → display toast
- Connection lost → reconnection attempt → failure screen
- Invalid action → reject with Spanish message

---

### Phase 12: Launch Prep (Week 23-24)

#### Task 12.1: Lobby & Matchmaking
**Files:** `client/scenes/lobby/`
- `Lobby.tscn` — Room browser, Quick Play, room code entry
- Room creation with mode selection (Quick/Standard/Long)
- Deep link support: `imperio://join/ROOMCODE`

#### Task 12.2: Results & Share
**Files:** `client/scenes/results/`
- `Results.tscn` — Final rankings, wealth breakdown
- `ShareCard.tscn` — Portada generation (per spec D2)
  - Player name + dynasty
  - Final rank, wealth, key stat
  - QR code / deep link
  - Export as PNG

#### Task 12.3: Asset Placeholders
- Property card images (25 unique)
- District map background
- UI elements (buttons, panels)
- Audio files (ambient + SFX)

> **Note:** AI-generated assets per Appendix G can be deferred. Use placeholders for testing.

#### Task 12.4: Performance Profiling
- Frame rate target: 60fps on mid-range mobile
- Memory usage < 512MB
- Network bandwidth < 10KB/s steady state
- Startup time < 5s

#### Task 12.5: Soft Launch Checklist
- [ ] Web export deployed to staging
- [ ] SSL configured and working
- [ ] 12-player stress test passed
- [ ] All Spanish strings from `locales_es.json`
- [ ] No placeholder text visible
- [ ] Analytics hooks (optional)

---

## Testing Strategy

### Server Module Testing

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

### Client Testing

| Component | Strategy | Verification |
|-----------|----------|--------------|
| Card Framework | Manual + GUT | Drag/drop works, containers layout correctly |
| ImperioClient | Integration | Connect, receive state, send messages |
| State Sync | Integration | Cards move on ownership change |
| Phase Panels | Manual | All phases navigable, inputs work |
| Visual Ledger | Manual | Animation completes, values correct |
| Mobile Input | Device test | Touch targets, no keyboard issues |
| Web Export | Browser test | Chrome, Firefox, Safari on desktop + mobile |

**Client Test Tools:**
- [GUT (Godot Unit Test)](https://github.com/bitwes/Gut) for GDScript unit tests
- Manual testing for UI/UX flows
- Browser DevTools for network debugging

### Test Commands
```bash
# Server: Single module
npm test -w @imperio/economy

# Server: All modules
npm test

# Server: Coverage
npm test -- --coverage

# Server: E2E only
npm test -w @imperio/room -- --testPathPattern=e2e

# Client: Run GUT tests (from Godot editor or CLI)
godot --headless -s addons/gut/gut_cmdln.gd -gexit
```

---

## Critical Files Reference

| File | Purpose |
|------|---------|
| `imperio_master_spec_v11.md` | Master spec (v12.4), single source of truth |
| `spec_amendments_v12_4.md` | Open source foundation commitments |
| `plans/open-source-foundation-research.md` | Code audit results for reuse decisions |
| `graf_dollar.txt` | Ledger schema and accounting rules |
| `data/properties.json` | 25-property deck |
| `data/events.json` | 52+ event deck |
| `data/moods.json` | 8 mood options |
| `data/signals.json` | 10 market signals |
| `data/teams.json` | 17 PARODY + 17 REAL dynasties |

### Open Source Foundations (Spec Appendix O)

| Component | Repository | Action |
|-----------|------------|--------|
| Card UI | `chun92/card-framework` | Copy addon to `client/addons/` |
| WebSocket/SSL | `alpapaydin/Godot4-Multiplayer` | Adapt patterns |
| Server | Colyseus official starter | `npm create colyseus-app` |

---

## Verification

### Server Verification
1. **Type Safety:** `tsc --noEmit` passes for all packages
2. **Unit Tests:** `npm test` all green, coverage thresholds met
3. **Integration:** Full 7-round match completes without errors
4. **Load Test:** 12 players, <100ms message latency at p99
5. **Spec Compliance:** Manual audit against spec formulas

### Client Verification
1. **Framework Integration:** card-framework addon works in Godot 4.5+
2. **Connection:** WebSocket connects to Colyseus, receives state
3. **State Sync:** PropertyCards update correctly on state changes
4. **UI Flow:** All phases navigable, inputs responsive
5. **Web Export:** Runs in Chrome/Firefox/Safari without errors
6. **Mobile:** Touch targets work, no input issues
7. **SSL:** `wss://` connection works in production

---

## Timeline Summary

| Phase | Scope | Weeks | Cumulative |
|-------|-------|-------|------------|
| 1-2 | Server Foundation + Economy | 1-4 | 4 weeks |
| 3 | Game Actions | 5-6 | 6 weeks |
| 4 | Events + Voting | 7-8 | 8 weeks |
| 5 | AI + Persistence | 9-10 | 10 weeks |
| 6 | Server Integration | 11-12 | 12 weeks |
| 7 | Client Foundation | 13-14 | 14 weeks |
| 8 | Card UI Integration | 15-16 | 16 weeks |
| 9 | Match UI | 17-18 | 18 weeks |
| 10 | Polish + Export | 19-20 | 20 weeks |
| 11 | Client-Server Integration | 21-22 | 22 weeks |
| 12 | Launch Prep | 23-24 | 24 weeks |

**Total:** ~24 weeks (6 months) from start to soft launch readiness.

---

## Execution Options

After approval:
1. **Subagent-Driven (this session)** — Fresh subagent per task, review between tasks
2. **Parallel Session (separate)** — Open new session with executing-plans skill
3. **Server-First** — Complete Phases 1-6, then start client in parallel
4. **Vertical Slice** — Build one complete feature (e.g., auction) end-to-end first
