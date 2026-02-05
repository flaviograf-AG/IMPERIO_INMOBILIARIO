# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**IMPERIO INMOBILIARIO** — viral multiplayer real estate tycoon game set in Lima, Peru. Players bid on properties, manage portfolios, deal with market shocks, and build dynasties in 7-14 round matches.

**Current Status**: Specification stage (v12.3 finalized). No source code yet — implementation pending.

## Technology Stack

| Layer | Technology |
|-------|------------|
| Server | Colyseus (authoritative state, rooms, WebSocket) on Node.js 20+ |
| Client | Godot 4.x (web export via WebAssembly) |
| Database | PostgreSQL (match snapshots as JSONB + Graf Dollar ledger) |
| Cache | Redis (optional: rate limiting, sessions) |
| Languages | TypeScript strict mode (server), GDScript/GodotJS (client) |

## Critical Files

| File | Purpose |
|------|---------|
| `imperio_master_spec_v11.md` | Master spec (v12.3) — ~6400 lines, use offset/limit to read |
| `graf_dollar.txt` | Double-entry ledger schema (PostgreSQL tables, idempotency keys) |
| `plans/modular-implementation-plan.md` | 14-package server architecture, testing strategy, task breakdown |
| `data/*.json` | Satellite game data (properties, events, signals, moods, teams, elections, quickchat, locales) |
| `spec_amendments_v12_*.md` | Changelog for spec versions 12.1-12.3 |
| `skills/gaming-design-bible/` | Skill for auditing game design against industry best practices |

## Planned Module Structure (Server)

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

Dependency rule: Modules can only depend on same or lower levels. No circular deps.

## Development Commands (Planned)

```bash
# Single module tests
npm test -w @imperio/economy

# All modules
npm test

# Coverage
npm test -- --coverage

# E2E tests only
npm test -w @imperio/room -- --testPathPattern=e2e

# Type checking
tsc --noEmit
```

## Master Spec Navigation

The spec (`imperio_master_spec_v11.md`, v12.3) is the single source of truth. Key sections:

| Section | What You'll Find |
|---------|------------------|
| Part 1 | Design philosophy, hard constraints |
| Part 2 | Game modes (Quick/Standard/Long), starting values (GD 500K/650K/800K) |
| Part 3 | Roles (Owner/Watcher), Quick Chat, dynasties, viral systems, challenges, streamer mode |
| Part 5 | Core loop, 11-phase state machine, Major/Minor action split, MOOD spec |
| Part 6 | Economy: GD-scale formulas (Wealth/Rent/OpsCost/Package bonuses), forced sales |
| Part 7 | Actions (Develop/Package/Insure/PR/etc.), 7 Deal templates, price validation |
| Part 8 | Targeted event system, RiskScore, fairness caps (150K max), 10s choice window |
| Part 13 | Graf Dollar Metagame: persistent wallet, leaderboard tiers, seasonal decay |
| Part 14 | Matchmaking: lobby browser, Quick Play, room codes, deep links, ELO ranked |
| A1 | TypeScript interfaces |
| A7 | 25-property deck with GD values (200K-2M range) |
| A8 | Bot heuristics H1-H9 |
| A9 | Colyseus + Godot integration examples |
| B | 52+ event deck |
| F | Networking protocol |
| V | Visual spec: "Tycoon Luxury" style, dynamic audio layers |

## Core Architecture

### State Machine — 11 Phases Per Round
```
REVEAL → BID → INCOME_CALC → VISUAL_LEDGER → ACTION → MOOD → DEALS → EVENTS → PRESS → [AWARDS] → SNAPSHOT
```
On election rounds, `VOTE` replaces `MOOD`. Full state: `LOBBY → MATCH_INIT → [rounds] → END`.

### Key Formulas (GD Scale)
```
Wealth = Cash + Σ(PropertyWorth) - Debt
PropertyWorth = BasePrice + (UpgradeLevel × 40,000) + WorthBonuses + PackageWorthBonus
Rent = BaseRent + (UpgradeLevel × 12,000) + DistrictSetBonus + SignalBonus + TempBonus + PackageRentBonus
OpsCost = floor((NumProperties + TotalUpgrades) / 3) × 12,000
LeaderTax = 5% of TotalRent (top quartile) | 10% of TotalRent (#1 if > 1.2x #2)
```

### Currency: Graf Dollar (GD)
- **In-match**: All players start equal (GD 500K/650K/800K by mode). Properties range GD 200K-2M.
- **Persistent**: Match final Wealth → lifetime wallet for leaderboard ranking (prestige-only, no in-match advantage).
- **Ledger**: Double-entry system per `graf_dollar.txt`. Tables: `ledger_tx`, `ledger_entry`, `balance_cache`.

### Security Model
- **JIT round seed**: `roundSeed = sha256(matchSeed + serverSecret + roundNumber)` — `serverSecret` NEVER sent to client
- **Idempotency**: All client messages include UUID `idempotencyKey`; server rejects duplicates
- **Phase gating**: Server rejects actions outside correct phase
- **Rate limits**: 1 bid/action/deal/vote per round, 1 rumor per match, 2 quick chats per phase

## Hard Constraints (Cannot Change)

| Constraint | Value |
|------------|-------|
| Max Owners | 12 (Standard mode) |
| Max Participants | 20 (Owners + Watchers) |
| Events | Targeted at single Owner (probability scales with properties/wealth/fame) |
| Insurance | Must have cost AND benefit |
| Catch-up mechanics | Mandatory (leader friction tax, fairness caps) |
| Deals | Template-only: 7 types, no free-form |
| Currency | Graf Dollar (GD) — prestige leaderboard, no in-match advantage |

## Language Rules

- **Player-facing content** (UI, cards, headlines): Spanish only
- **Code** (variables, comments, types): English only
- Spanish strings loaded from `data/locales_es.json`

## Economic Tags (closed set)
`turismo`, `oficinas`, `logistica`, `residencial`, `gobierno`, `cultura`, `costa`

Flavor tags: `premium_area`, `corredor_obra`, `zona_caliente`

## Available Skills

- `gaming-design-bible` — Use when designing game features, balancing economy, or auditing spec against industry best practices.

## Spec Amendment History

| Version | File | Scope |
|---------|------|-------|
| v12.1 | `spec_amendments_v12_1.md` | Critical/High: OpsCost rebalance, leader tax, matchmaking, challenges |
| v12.2 | `spec_amendments_v12_2.md` | Medium: Property deck fixes, 8 new events, TypeScript interfaces |
| v12.3 | `spec_amendments_v12_3.md` | Low: Adaptive Visual Ledger timing, dynamic audio layers |
