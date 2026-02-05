# Open Source Foundation Research

**Date:** 2026-02-05
**Status:** Phase 2 Complete (code audit done for top 3 candidates)
**Purpose:** Evaluate open source repositories to maximize code reuse for Imperio Inmobiliario

---

## Executive Summary

**Key Finding:** No single repository combines Colyseus + Godot 4 + turn-based board game.

**Reuse Estimate:**
- Infrastructure code: ~50-60% reusable
- Game logic: 0% reusable (must be custom per spec)

**Top Recommendations:**
1. **chun92/card-framework** — Use for card UI (MIT, Godot 4.5+, excellent quality)
2. **alpapaydin/Godot4-Multiplayer-Survivor-IO-Game** — Use for WebSocket/SSL patterns (MIT, Godot 4.3)
3. **Skip sominator/colyseus-2d-card-game-templates** — UNLICENSED, Godot 3.x, minimal value

---

## Detailed Audit Results

### 1. sominator/colyseus-2d-multiplayer-card-game-templates

**URL:** https://github.com/sominator/colyseus-2d-multiplayer-card-game-templates
**Verdict:** **NOT RECOMMENDED**

| Criteria | Finding | Impact |
|----------|---------|--------|
| **License** | UNLICENSED (package.json) vs MIT (README) | **BLOCKER** — Legal ambiguity |
| **Godot Version** | 3.x (`config_version=4`, `yield`, `WebSocketClient`) | **Major port needed** |
| **Colyseus Version** | 0.14.9 | Old (current is 0.15+) |
| **Server Code** | 54 lines, no game logic, just message echo | **No value** |
| **Game Logic** | None — "Hello World" level | **No value** |
| **Activity** | 2 commits total, last update 2023 | **Abandoned** |

**Code Analysis:**
- Server: `maxClients = 2` (not 12), no phases, no state machine
- Client: Godot 3.x syntax (`yield`, `funcref`, `PoolByteArray`)
- Colyseus addon: Would need 200+ line changes for Godot 4.x

**Conclusion:** Skip entirely. The effort to port exceeds starting fresh.

---

### 2. alpapaydin/Godot4-Multiplayer-Survivor-IO-Game

**URL:** https://github.com/alpapaydin/Godot4-Multiplayer-Survivor-IO-Game
**Verdict:** **RECOMMENDED FOR PATTERNS**

| Criteria | Finding | Impact |
|----------|---------|--------|
| **License** | MIT (2024) | **CLEAR** — Can use freely |
| **Godot Version** | 4.3 (`config_version=5`) | **Current** — No porting needed |
| **Architecture** | Godot built-in multiplayer (not Colyseus) | **Different from spec** |
| **Server Model** | Authoritative | **Excellent** — Matches our needs |
| **SSL/TLS** | Full Let's Encrypt integration | **Production-ready** |
| **Export Presets** | Web + Linux dedicated server | **Complete pipeline** |
| **Activity** | 76 commits, comprehensive README | **Active, documented** |

**Architecture Note:** Uses WebSocketMultiplayerPeer + RPC, not Colyseus. Our spec calls for Colyseus (TypeScript server), so this is pattern reference only.

**Reusable Components:**
```
HIGH VALUE (directly adaptable):
├── WebSocket SSL/TLS setup (Multihelper.gd:36-66)
├── Export presets (export_presets.cfg) — Web + Linux server
├── Constants pattern (Constants.gd) — Config externalization
├── Chat UI (message_box.gd) — Animated floating messages
├── Player connection flow (Multihelper.gd:92-121)
└── Server/client detection patterns (multiplayer.is_server())

MEDIUM VALUE (study patterns):
├── RPC decorators → Convert to Colyseus messages
├── Player spawn/despawn → Adapt for Colyseus schema
└── Scene management (Game.gd) — Server-authoritative level changes
```

**Key Code Patterns:**

SSL WebSocket Connection:
```gdscript
func join_game(address = ""):
    var peer = WebSocketMultiplayerPeer.new()
    if Constants.USE_SSL:
        var cert := load(Constants.TRUSTED_CHAIN_PATH)
        var tlsOptions = TLSOptions.client(cert)
        error = peer.create_client("wss://" + address + ":" + str(PORT), tlsOptions)
```

**Conclusion:** Extract WebSocket/SSL patterns, export presets, and chat UI. Don't adopt Godot multiplayer — keep Colyseus per spec.

---

### 3. chun92/card-framework

**URL:** https://github.com/chun92/card-framework
**Verdict:** **HIGHLY RECOMMENDED**

| Criteria | Finding | Impact |
|----------|---------|--------|
| **License** | MIT (2025) | **CLEAR** — Can use freely |
| **Godot Version** | 4.5+ (`config_version=5`) | **Current** — Ready to use |
| **Documentation** | Excellent (API.md, GETTING_STARTED.md) | **Production-ready** |
| **Code Quality** | Clean, well-documented, proper patterns | **High quality** |
| **Architecture** | Factory pattern, inheritance, events | **Extensible** |
| **Multiplayer** | **Not included** | Needs network integration |
| **Last Update** | January 2026 (v1.3.1) | **Actively maintained** |

**What It Provides:**
```
DIRECTLY REUSABLE:
├── Card class — Front/back faces, hover effects, drag states
├── CardContainer base — Collection management, undo history
├── Pile — Stack layout (face-up/face-down)
├── Hand — Fanned layout (player hand)
├── DraggableObject — Mouse interaction state machine
├── DropZone — Drag target validation
├── JsonCardFactory — JSON-based card creation
└── CardManager — Orchestrates all cards and containers
```

**Imperio Mapping:**
| Framework Component | Imperio Use Case |
|---------------------|------------------|
| Card | Property cards |
| Hand | Player portfolio |
| Pile | Market row, event deck |
| CardManager | Match scene orchestrator |
| JsonCardFactory | Load properties.json |

**Multiplayer Integration Pattern:**
```gdscript
# When Colyseus state changes, update card positions
func _on_colyseus_state_changed(state):
    for property_id in state.marketRow:
        var card = card_manager.get_card(property_id)
        market_pile.add_card(card)
```

**Conclusion:** Use as primary UI foundation. Copy addon to project, extend for network sync.

---

## Other Candidates (Not Audited in Detail)

### boardgame.io
- **URL:** https://github.com/boardgameio/boardgame.io
- **Value:** Phase system patterns (12.2k stars, MIT)
- **Issue:** React-centric, different architecture than Colyseus
- **Action:** Study phase/stage architecture for inspiration

### gsioteam/godot-colyseus
- **URL:** https://github.com/gsioteam/godot-colyseus
- **Value:** Colyseus SDK for Godot
- **Issue:** Unclear Godot 4.x compatibility, low activity (25 commits)
- **Action:** Test for Godot 4 before using; may need to write custom WebSocket client

### rametta/Pali
- **URL:** https://github.com/rametta/Pali
- **Value:** Dedicated server pattern, point calculation (Apache 2.0)
- **Issue:** 2-player only, 3D focused
- **Action:** Reference for server pattern

---

## Strategic Recommendations

### Recommended Stack

```
Server (TypeScript):
├── Framework: Colyseus 0.15+ (per spec)
├── Database: PostgreSQL (per spec)
├── Patterns: Study boardgame.io phases
└── Start: Official Colyseus starter (npm create colyseus-app)

Client (Godot 4.5+):
├── Card UI: chun92/card-framework (copy addon)
├── WebSocket: Custom client to Colyseus (not Godot multiplayer)
├── SSL/TLS: Patterns from alpapaydin repo
├── Export: Presets from alpapaydin repo
└── Networking: Adapt spec A9 ImperioClient example
```

### What to Build vs. Reuse

| Component | Source | Notes |
|-----------|--------|-------|
| Card UI (drag, hover, layout) | chun92/card-framework | Copy and extend |
| WebSocket/SSL setup | alpapaydin patterns | Adapt for Colyseus |
| Export presets | alpapaydin | Copy directly |
| Colyseus TypeScript server | Official starter | Customize |
| 11-phase state machine | Custom | No existing match |
| Economy formulas | Custom | Spec-specific |
| Bot heuristics | Custom | Spec-specific |
| Event targeting | Custom | Spec-specific |
| Graf Dollar ledger | Custom | Per graf_dollar.txt |
| Visual Ledger animation | Custom | Spec-specific |

### Architecture Decision: Colyseus vs Godot Multiplayer

The spec calls for Colyseus (TypeScript server). The alpapaydin repo uses Godot's built-in multiplayer. Options:

| Option | Pros | Cons |
|--------|------|------|
| **A: Keep Colyseus** | TypeScript server (testable), schema validation, matches spec | Need custom Godot client |
| **B: Godot Multiplayer** | Simpler, alpapaydin ready-to-use | All GDScript (harder to test complex logic) |

**Recommendation:** Keep Colyseus per spec. The 11-phase state machine, economy formulas, and bot heuristics benefit from TypeScript's type safety and testability.

---

## Next Steps

1. **Spec Amendment v12.4** — Commit to these foundations:
   - Card UI: chun92/card-framework
   - WebSocket patterns: alpapaydin (reference)
   - Server: Colyseus official starter

2. **Expanded Implementation Plan** — Add client phases using these foundations

3. **Prototype** — Build minimal viable connection:
   - Colyseus server with single property schema
   - Godot client with card-framework showing property
   - WebSocket sync between them

---

## License Summary

| Repository | License | Commercial Use |
|------------|---------|----------------|
| chun92/card-framework | MIT | Yes |
| alpapaydin/Godot4-Multiplayer | MIT | Yes |
| sominator/colyseus-card-game | UNLICENSED | No (skip) |
| boardgame.io | MIT | Yes |
| gsioteam/godot-colyseus | MIT | Yes |
| rametta/Pali | Apache 2.0 | Yes |

---

## Appendix: Key Files from Audited Repos

### From alpapaydin (WebSocket/SSL patterns)
- `scenes/autoloads/Multihelper.gd` — WebSocket connection with TLS
- `scenes/autoloads/Constants.gd` — Server configuration
- `export_presets.cfg` — Web + Linux server exports

### From chun92 (Card UI)
- `addons/card-framework/card.gd` — Card class (165 lines)
- `addons/card-framework/card_container.gd` — Base container (428 lines)
- `addons/card-framework/pile.gd` — Stack layout
- `addons/card-framework/hand.gd` — Fanned layout
- `addons/card-framework/card_manager.gd` — Orchestrator
- `addons/card-framework/draggable_object.gd` — Mouse FSM
