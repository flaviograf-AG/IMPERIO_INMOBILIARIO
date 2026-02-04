# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**IMPERIO INMOBILIARIO** — viral multiplayer real estate tycoon game set in Lima, Peru. Players bid on properties, manage portfolios, deal with market shocks, and build dynasties in 7-14 round matches.

**Current Status**: Specification stage (v11.2 finalized). No source code yet — implementation pending.

## Technology Stack

| Layer | Technology |
|-------|------------|
| Server | Colyseus (authoritative state, rooms, WebSocket) on Node.js 20+ |
| Client | Godot 4.x (web export via WebAssembly) |
| Database | PostgreSQL (match snapshots as JSONB) |
| Cache | Redis (optional: rate limiting, sessions) |
| Languages | TypeScript strict mode (server), GDScript/GodotJS (client) |

## Master Spec Navigation

The spec (`imperio_master_spec_v11.md`, 4,591 lines) is the single source of truth. Use these line ranges to jump directly:

| Lines | Section | What You'll Find |
|-------|---------|------------------|
| 30-74 | Part 1 | Design philosophy, hard constraints |
| 76-158 | Part 2 | Game modes (Quick/Standard/Long), starting values |
| 160-252 | Part 3 | Roles (Owner/Watcher), identity, viral systems |
| 298-547 | Part 5 | **Core loop, 11-phase state machine, all phase details** |
| 550-629 | Part 6 | Economy: Wealth/Rent/OpsCost formulas, forced sales |
| 631-755 | Part 7 | Actions (Develop/Insure/PR/etc.) and Deal templates |
| 756-886 | Part 8 | Targeted event system, RiskScore, fairness caps |
| 930-975 | Part 10 | Press system, headline generation |
| 1011-1067 | Part 11 | Server architecture, DB tables, infra setup |
| 1069-1127 | Part 12 | Security: phase gating, validation, rate limits, reconnection |
| 1134-1332 | **A1** | **TypeScript interfaces — authoritative types** |
| 1334-1355 | A2 | Deal engine rules (deterministic resolution) |
| 1356-1389 | A3 | Full state machine diagram |
| 1486-1542 | A8 | Bot heuristics (tutorial/autopilot) |
| 1543-1688 | **A9** | **Colyseus + Godot integration code examples** |
| 1691-2358 | B | Complete 52+ event deck (YAML) with headlines |
| 2360-2445 | C | UI acceptance criteria, Visual Ledger animation spec |
| 2548-2656 | F | Networking protocol: envelope structure, message types, delta packets |
| 2659-3163 | **V** | **Visual spec: "Tycoon Luxury" style, UI kit, map, typography, colors** |
| 3166-3584 | O | Open source foundation evaluation |
| 3586-4591 | G | AI asset generation pipeline, prompts, batch workflows |

## Core Architecture

### State Machine — 11 Phases Per Round
```
REVEAL → BID → INCOME_CALC → VISUAL_LEDGER → ACTION → MOOD → DEALS → EVENTS → PRESS → [AWARDS] → SNAPSHOT
```
On election rounds, `VOTE` replaces `MOOD`. Full state: `LOBBY → MATCH_INIT → [rounds] → END`.

### Hard Ordering Rule (Part 5.3)
1. Resolve bids (auction winners)
2. Apply Income (rent + signals + ops + debt + leader tax)
3. Visual Ledger animation (4.0s, client-side, server waits)
4. Collect actions
5. Resolve Mood/Vote (queued effects)
6. Resolve deals (post-action ownership is authoritative)
7. Select targets and resolve events (apply fairness caps)
8. Generate headlines
9. Emit achievements → Emit snapshots

Server proceeds regardless of client response. Timeouts: no bid = no bid, no action = Pass, no vote = not counted.

### Key Formulas
```
Wealth = Cash + Σ(PropertyWorth) − Debt
PropertyWorth = BasePrice + (UpgradeLevel × 2) + WorthBonuses
Rent = BaseRent + (UpgradeLevel × 2) + DistrictSetBonus + MarketSignalBonus + TempRentBonus
OpsCost = floor((NumProperties + TotalUpgrades) / 4)
```
District set bonus: +1 rent per property when 2+ owned in same district.

### Satellite Data Strategy
Game configuration externalized to `data/` JSON files — allows balance updates without client rebuild. Files: `events.json` (52+ events), `signals.json` (10 market signals), `elections.json` (3 candidates), `teams.json` (17 PARODY + 17 REAL_LABEL dynasties), `locales_es.json` (Spanish UI strings).

### Security Model
- **JIT round seed**: `roundSeed = sha256(matchSeed + serverSecret + roundNumber)` — `serverSecret` NEVER sent to client
- **Idempotency**: All client messages include UUID `idempotencyKey`; server rejects duplicates
- **Phase gating**: Server rejects actions outside correct phase
- **Rate limits**: 1 bid/action/deal/vote per round, 1 rumor per match, 5 reconnect attempts/min

## Hard Constraints (Cannot Change)

| Constraint | Value |
|------------|-------|
| Max Owners | 12 (Standard mode) |
| Max Participants | 20 (Owners + Watchers) |
| Events | Targeted at single Owner (probability scales with properties/wealth/fame) |
| Insurance | Must have cost AND benefit |
| Catch-up mechanics | Mandatory (leader friction tax, fairness caps) |
| Deals | Template-only: 6 types, no free-form |
| Graf Design System | Admin UI only |
| Public rooms | Viewable; profiles unlisted by default |

## Language Rules

- **Player-facing content** (UI, cards, headlines): Spanish only
- **Code** (variables, comments, types): English only
- Spanish strings loaded from `data/locales_es.json`

## Economic Tags (closed set)
`turismo`, `oficinas`, `logistica`, `residencial`, `gobierno`, `cultura`, `costa`

Flavor tags: `premium_area`, `corredor_obra`, `zona_caliente`

## Implementation Patterns

### Colyseus State Schema
Use `@colyseus/schema` decorators. Key schemas: `OwnerSchema`, `PropertySchema`, `ImperioState`. See spec A9 (line 1587) for complete examples.

### Colyseus Room
`ImperioRoom extends Room<ImperioState>` with `maxClients = 20`. Message handlers: `"bid"`, `"action"`. Reconnection: allow 60s, then autopilot via bot heuristics (A8).

### Godot Client
WebSocket connection to Colyseus. Signals: `state_changed`, `phase_changed`. Handles `STATE_SNAPSHOT`, `PHASE_CHANGE`, and `OP_DELTA` message types. Mobile text inputs use HTML DOM overlays (not Godot virtual keyboard).

### Networking Protocol (Appendix F)
Envelope: `{ type, payload, ts, idempotencyKey }`. Delta packets (`OP_DELTA`) use `{ op, id, val }` format for efficient state updates.

## Visual Style

"Tycoon Luxury" — semi-realistic Lima cityscape, high-contrast UI with gold/bronze metallics. Full visual spec in Appendix V (line 2659+).
