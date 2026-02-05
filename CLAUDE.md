# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**IMPERIO INMOBILIARIO** — viral multiplayer real estate tycoon game set in Lima, Peru. Players bid on properties, manage portfolios, deal with market shocks, and build dynasties in 7-14 round matches.

**Current Status**: Specification stage (v12.1 finalized). No source code yet — implementation pending.

## Technology Stack

| Layer | Technology |
|-------|------------|
| Server | Colyseus (authoritative state, rooms, WebSocket) on Node.js 20+ |
| Client | Godot 4.x (web export via WebAssembly) |
| Database | PostgreSQL (match snapshots as JSONB + Graf Dollar ledger) |
| Cache | Redis (optional: rate limiting, sessions) |
| Languages | TypeScript strict mode (server), GDScript/GodotJS (client) |

## Master Spec Navigation

The spec (`imperio_master_spec_v11.md`, v12.1) is the single source of truth. Key sections:

| Section | What You'll Find |
|---------|------------------|
| Part 1 | Design philosophy, hard constraints |
| Part 2 | Game modes (Quick/Standard/Long), starting values (GD 500K/650K/800K) |
| Part 3 | Roles (Owner/Watcher), Quick Chat, **dynasties (social features)**, viral systems, **challenges**, **streamer mode**, leaderboard |
| Part 5 | **Core loop, 11-phase state machine, Major/Minor action split, MOOD full spec** |
| Part 6 | Economy: GD-scale formulas (Wealth/Rent/OpsCost/Package bonuses), forced sales, **late-game cash sinks** |
| Part 7 | Actions (Develop levels 1-5/Package/Insure/PR/etc.), 7 Deal templates + **price validation**, Smart Suggestions, Property Packaging |
| Part 8 | Targeted event system, RiskScore, GD fairness caps (150K max), **10s player choice window** |
| Part 13 | **Graf Dollar Metagame: persistent wallet, leaderboard tiers, seasonal decay, dynasty rankings** |
| Part 14 | **Matchmaking: lobby browser, Quick Play, room codes, deep links, bot-fill rules, ranked ELO** |
| A1 | TypeScript interfaces — PropertyPackage, MoodOption, QuickChatMessage types |
| A7 | **25-property deck with GD values (200K-2M range)** |
| A8 | Bot heuristics with GD thresholds **(H1-H9 + standardized reserve)** |
| A9 | Colyseus + Godot integration code examples |
| B | Complete 52+ event deck with GD cash amounts |
| F | Networking protocol: PACKAGE, QUICK_CHAT, SELL_PACKAGE messages |
| V | Visual spec: "Tycoon Luxury" style, GD currency bills |

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
4. Collect actions (1 Major + 0-1 Minor)
5. Resolve Mood/Vote (queued effects)
6. Resolve deals (post-action ownership is authoritative)
7. Select targets and resolve events (apply fairness caps)
8. Generate headlines
9. Emit achievements → Emit snapshots

Server proceeds regardless of client response. Timeouts: no bid = no bid, no action = Pass, no vote = not counted.

### Key Formulas (GD Scale)
```
Wealth = Cash + Σ(PropertyWorth) - Debt
PropertyWorth = BasePrice + (UpgradeLevel × 40,000) + WorthBonuses + PackageWorthBonus
Rent = BaseRent + (UpgradeLevel × 12,000) + DistrictSetBonus + SignalBonus + TempBonus + PackageRentBonus
OpsCost = floor((NumProperties + TotalUpgrades) / 3) × 12,000
LeaderTax = 5% of TotalRent (top quartile) | 10% of TotalRent (#1 if > 1.2x #2)
```
District set bonus: +GD 5,000 rent per property when 2+ owned in same district.

### Currency: Graf Dollar (GD)
- **In-match**: All players start equal (GD 500K/650K/800K by mode). Properties range GD 200K-2M.
- **Persistent**: Match final Wealth → lifetime wallet for leaderboard ranking (prestige-only, no in-match advantage).
- **Ledger**: Double-entry system per `graf_dollar.txt`. Tables: `ledger_tx`, `ledger_entry`, `balance_cache`.

### Satellite Data Strategy
Game configuration externalized to `data/` JSON files — allows balance updates without client rebuild. Files: `properties.json` (25 properties), `events.json` (52+ events), `signals.json` (10 market signals), `elections.json` (4 candidates), `moods.json` (8 mood options), `quickchat.json` (12 presets), `teams.json` (17 PARODY + 17 REAL_LABEL dynasties), `locales_es.json` (Spanish UI strings), `challenges.json` (daily/weekly challenges).

### Security Model
- **JIT round seed**: `roundSeed = sha256(matchSeed + serverSecret + roundNumber)` — `serverSecret` NEVER sent to client
- **Idempotency**: All client messages include UUID `idempotencyKey`; server rejects duplicates
- **Phase gating**: Server rejects actions outside correct phase
- **Rate limits**: 1 bid/action/deal/vote per round, 1 rumor per match, 2 quick chats per phase, 5 reconnect attempts/min

## Hard Constraints (Cannot Change)

| Constraint | Value |
|------------|-------|
| Max Owners | 12 (Standard mode) |
| Max Participants | 20 (Owners + Watchers) |
| Events | Targeted at single Owner (probability scales with properties/wealth/fame) |
| Insurance | Must have cost AND benefit |
| Catch-up mechanics | Mandatory (leader friction tax, fairness caps) |
| Deals | Template-only: 7 types, no free-form |
| Currency | Graf Dollar (GD) — prestige leaderboard, no in-match advantage from persistent wallet |
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
Use `@colyseus/schema` decorators. Key schemas: `OwnerSchema`, `PropertySchema`, `ImperioState`. See spec A9 for complete examples.

### Colyseus Room
`ImperioRoom extends Room<ImperioState>` with `maxClients = 20`. Message handlers: `"bid"`, `"action"`, `"package"`, `"quick_chat"`. Reconnection: allow 60s, then autopilot via bot heuristics (A8).

### Godot Client
WebSocket connection to Colyseus. Signals: `state_changed`, `phase_changed`. Handles `STATE_SNAPSHOT`, `PHASE_CHANGE`, `OP_DELTA`, `QUICK_CHAT_MSG`, `PACKAGE_CREATED` message types. Mobile text inputs use HTML DOM overlays (not Godot virtual keyboard).

### Networking Protocol (Appendix F)
Envelope: `{ type, payload, ts, idempotencyKey }`. Delta packets (`OP_DELTA`) use `{ op, id, val }` format. New ops: `SET_PACKAGE`, `ADD_PACKAGE`, `REMOVE_PACKAGE`.

## Visual Style

"Tycoon Luxury" — semi-realistic Lima cityscape, high-contrast UI with gold/bronze metallics. Graf Dollar bills in 3 denominations (GD 50K, 100K, 500K). Full visual spec in Appendix V.
