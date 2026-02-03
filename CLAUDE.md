# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**IMPERIO INMOBILIARIO** is a viral multiplayer real estate tycoon game set in Lima, Peru. Players bid on properties, manage portfolios, deal with market shocks, and build dynasties in competitive 7-14 round matches.

**Current Status**: Specification stage (v11.2 finalized). No source code yet—implementation pending.

## Technology Stack

| Layer | Technology |
|-------|------------|
| Server | Colyseus (authoritative state, rooms, WebSocket) |
| Client | Godot 4.x (web export via WebAssembly) |
| Database | PostgreSQL |
| Cache | Redis (optional, rate limiting) |
| Languages | TypeScript (server), GDScript/GodotJS (client) |

## Project Structure

```
IMPERIO_INMOBILIARIO/
├── data/                    # External game configuration (Satellite Data)
│   ├── elections.json       # 3 election candidates with effects
│   ├── events.json          # 52+ events with conditional logic
│   ├── locales_es.json      # Spanish UI strings
│   ├── signals.json         # 10 market signals affecting rent
│   └── teams.json           # 17 PARODY + 17 REAL_LABEL dynasties
├── legacy/                  # Archived spec versions (v9, v10)
└── imperio_master_spec_v11.md  # ACTIVE MASTER SPECIFICATION (4,591 lines)
```

## Master Specification Reference

The spec (`imperio_master_spec_v11.md`) is the authoritative source for all game rules. Key sections:

| Section | Content |
|---------|---------|
| Part 1 | Design philosophy, hard constraints |
| Part 2 | Game modes (Quick/Standard/Long) |
| Part 5 | Core loop, 11-phase state machine |
| Appendix A1 | TypeScript interfaces (authoritative types) |
| Appendix A9 | Colyseus/Godot integration examples |
| Appendix O | Open source foundation evaluation |

## Core Architecture

### Phase Sequence (11 phases per round)
```
REVEAL → BID → INCOME_CALC → VISUAL_LEDGER → ACTION → MOOD → DEALS → EVENTS → PRESS → [AWARDS] → SNAPSHOT
```

On election rounds, `VOTE` replaces `MOOD`.

### Satellite Data Strategy
Game configuration externalized to `/data/` JSON files. Allows live balance updates without client rebuild.

### Security Model
```typescript
// JIT round seed - prevents client prediction
roundSeed = sha256(matchSeed + serverSecret + roundNumber)
```
`serverSecret` never sent to client.

## Hard Constraints (Cannot Change)

| Constraint | Value |
|------------|-------|
| Max Owners | 12 (Standard mode) |
| Max Participants | 20 (Owners + Watchers) |
| Events | Targeted at single Owner |
| Insurance | Must have cost AND benefit |
| Catch-up mechanics | Mandatory (fairness caps) |
| Deals | Template-only (6 types, no free-form) |

## Language Rules

- **Player-facing content** (UI, cards, headlines): Spanish only
- **Code** (variables, comments, types): English only
- Spanish strings loaded from `data/locales_es.json`

## Data File Schemas

### events.json
```json
{
  "id": "E-045",
  "title_es": "Incendio",
  "type": "INCIDENT",           // GLOBAL_TARGET | TARGETED | INCIDENT
  "condition": "...",           // Optional gate (e.g., DISTRICT_HAS_INFRA)
  "effects": { "cash": -3, "rentDelta": -2 },
  "mitigation": { "INSURANCE_BASIC": {...} },
  "fallback": "E-045B"          // Optional fallback event
}
```

### signals.json
```json
{ "id": "S-01", "name": "Boom Airbnb", "effects": { "turismo": 1, "costa": 1 } }
```

### Economic Tags (closed set)
`turismo`, `oficinas`, `logistica`, `residencial`, `gobierno`, `cultura`, `costa`

## Implementation Notes

### Colyseus Room Setup
```typescript
// server/src/rooms/ImperioRoom.ts
export class ImperioRoom extends Room<ImperioState> {
  maxClients = 20;
  // Message handlers: "bid", "action"
}
```

### State Schema Pattern
```typescript
// Use @colyseus/schema decorators
@type("number") cash: number = 10;
@type({ map: OwnerSchema }) owners = new MapSchema<OwnerSchema>();
```

### Godot Client Signals
```gdscript
signal state_changed(state)
signal phase_changed(phase)
```

## Key Game Modes

| Mode | Rounds | Timer | Players |
|------|--------|-------|---------|
| Quick (viral default) | 7 | 25s | 4-12 |
| Standard | 10 | 35s | 4-12 |
| Long | 14 | 40s | 4-8 |

## Deal Templates (6 types only)
`CASH_FOR_PROPERTY`, `SWAP`, `BANK_LOAN`, `ELECTION_PACT`, `NON_COMPETE`, `SECURITY_ESCORT`

## Visual Style
"Tycoon Luxury" — semi-realistic Lima cityscape, high-contrast UI with gold/bronze metallics.
