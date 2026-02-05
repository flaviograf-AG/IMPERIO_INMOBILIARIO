# Spec Amendment v12.4: Open Source Foundation Commitments

**Version:** 12.4
**Date:** 2026-02-05
**Status:** APPLIED to master spec
**Scope:** Commits to specific open source foundations based on code audit results

---

## Summary

This amendment converts Appendix O from an "evaluation matrix" to a "foundation directive" — specifying exactly which open source repositories to use and how to integrate them.

**Key Decisions:**
- **Card UI:** Use `chun92/card-framework` (MIT, Godot 4.5+)
- **WebSocket/SSL Patterns:** Extract from `alpapaydin/Godot4-Multiplayer-Survivor-IO-Game` (MIT)
- **Server:** Colyseus official starter (per existing spec)
- **Skip:** `sominator/colyseus-2d-card-game-templates` (UNLICENSED, Godot 3.x)

---

## Changes to Section 1.5: Open Source Foundation Strategy

### REPLACE entire section 1.5 with:

## 1.5 Open Source Foundation Commitments

This project uses the following proven open-source foundations. These are **directives**, not recommendations.

### 1.5.1 Server Foundation
- **Framework:** Colyseus 0.15+ via `npm create colyseus-app@latest`
- **Language:** TypeScript strict mode
- **Rationale:** Authoritative state sync, room architecture, delta compression

### 1.5.2 Client Foundation
- **Engine:** Godot 4.5+ (GL Compatibility renderer for WebGL2)
- **Card UI:** `chun92/card-framework` addon (copy to `addons/card-framework/`)
- **WebSocket:** Custom client adapting patterns from `alpapaydin/Godot4-Multiplayer` for SSL/TLS
- **Rationale:** MIT-licensed, actively maintained, Godot 4.5+ compatible

### 1.5.3 What We Reuse vs. Build Custom

| Component | Source | Custom Work Required |
|-----------|--------|---------------------|
| Card drag/drop UI | chun92/card-framework | Extend for network sync |
| Card containers (Pile, Hand) | chun92/card-framework | Map to Colyseus schema |
| WebSocket SSL/TLS setup | alpapaydin patterns | Adapt for Colyseus protocol |
| Export presets (Web + Linux) | alpapaydin | Copy directly |
| Colyseus server scaffold | Official starter | Add game logic |
| 11-phase state machine | Custom | No existing match |
| Economy formulas | Custom | Spec-specific |
| Bot heuristics H1-H9 | Custom | Spec-specific |
| Event targeting/RiskScore | Custom | Spec-specific |
| Graf Dollar ledger | Custom | Per `graf_dollar.txt` |
| Visual Ledger animation | Custom | Spec-specific |
| Share card (Portada) | Custom | Spec-specific |

---

## Changes to Section 11.2: Client Architecture (Godot 4.x)

### REPLACE section 11.2 with expanded version:

## 11.2 Client Architecture (Godot 4.5+)

### 11.2.1 Engine Configuration
```
Godot Version: 4.5+
Renderer: GL Compatibility (WebGL2 for browser support)
Export Targets: Web (primary), Linux (dedicated server for development)
```

### 11.2.2 Project Structure
```
client/
├── project.godot
├── addons/
│   └── card-framework/        # Copy from chun92/card-framework
│       ├── card.gd
│       ├── card_container.gd
│       ├── pile.gd
│       ├── hand.gd
│       ├── card_manager.gd
│       ├── draggable_object.gd
│       └── ...
├── autoloads/
│   ├── Constants.gd           # Server URL, SSL config
│   ├── ImperioClient.gd       # Colyseus WebSocket client
│   └── GameState.gd           # Local state mirror
├── scenes/
│   ├── lobby/
│   │   ├── Lobby.tscn
│   │   └── lobby.gd
│   ├── match/
│   │   ├── Match.tscn         # Main match scene
│   │   ├── match.gd
│   │   ├── phases/            # Phase-specific UI panels
│   │   │   ├── BidPanel.tscn
│   │   │   ├── ActionPanel.tscn
│   │   │   ├── DealPanel.tscn
│   │   │   └── ...
│   │   ├── components/
│   │   │   ├── PropertyCard.tscn  # Extends card-framework Card
│   │   │   ├── MarketRow.tscn     # Extends Pile
│   │   │   ├── Portfolio.tscn     # Extends Hand
│   │   │   ├── VisualLedger.tscn
│   │   │   ├── QuickChat.tscn
│   │   │   └── EventOverlay.tscn
│   │   └── hud/
│   │       ├── CashDisplay.tscn
│   │       ├── PhaseTimer.tscn
│   │       └── PlayerList.tscn
│   └── results/
│       ├── Results.tscn
│       ├── results.gd
│       └── ShareCard.tscn     # Portada generation
├── data/                      # Satellite JSON (synced from server)
│   ├── properties.json
│   ├── events.json
│   └── ...
└── assets/
    ├── cards/                 # Property card images
    ├── ui/                    # UI elements
    ├── audio/                 # SFX and ambient
    └── certs/                 # SSL certificates for production
```

### 11.2.3 Card Framework Integration

The `chun92/card-framework` addon provides:
- **Card** — Base card with front/back faces, hover effects, drag states
- **CardContainer** — Base container with undo history
- **Pile** — Stack layout (for market row, event deck)
- **Hand** — Fanned layout (for player portfolio)
- **CardManager** — Orchestrates all cards and containers
- **DraggableObject** — Mouse interaction state machine (IDLE → HOVERING → HOLDING)

**Imperio Extensions:**

```gdscript
# scenes/match/components/PropertyCard.gd
class_name PropertyCard
extends Card

# Imperio-specific properties synced from Colyseus
var property_id: String
var owner_id: String
var upgrade_level: int = 0
var is_insured: bool = false
var package_id: String = ""

# Display GD values
func update_from_state(property_state: Dictionary):
    property_id = property_state.get("propertyId", "")
    owner_id = property_state.get("ownerId", "")
    upgrade_level = property_state.get("upgradeLevel", 0)
    is_insured = property_state.get("isInsured", false)
    # Update visual indicators
    _update_upgrade_indicator()
    _update_insurance_badge()
```

```gdscript
# scenes/match/components/MarketRow.gd
class_name MarketRow
extends Pile

signal property_selected(property_id: String)

func _on_card_pressed(card: Card):
    if card is PropertyCard:
        property_selected.emit(card.property_id)
```

### 11.2.4 WebSocket Client with SSL

Adapted from `alpapaydin/Godot4-Multiplayer-Survivor-IO-Game` patterns:

```gdscript
# autoloads/Constants.gd
extends Node

const SERVER_IP := "localhost"
const PORT := 2567
const USE_SSL := false  # Set true for production
const CERT_PATH := "res://assets/certs/fullchain.pem"
```

```gdscript
# autoloads/ImperioClient.gd
extends Node

signal connected
signal disconnected
signal state_changed(state: Dictionary)
signal phase_changed(phase: String, ends_at: int)
signal error_received(code: String, message: String)

var websocket: WebSocketPeer
var is_connected: bool = false

func connect_to_server():
    websocket = WebSocketPeer.new()
    var url: String

    if Constants.USE_SSL:
        url = "wss://%s:%d" % [Constants.SERVER_IP, Constants.PORT]
        # Note: For web exports, browser handles SSL validation
        # For native builds, load certificate:
        # var cert = load(Constants.CERT_PATH)
    else:
        url = "ws://%s:%d" % [Constants.SERVER_IP, Constants.PORT]

    var error = websocket.connect_to_url(url)
    if error != OK:
        push_error("WebSocket connection failed: %d" % error)
        return error
    return OK

func _process(_delta):
    if websocket == null:
        return

    websocket.poll()
    var state = websocket.get_ready_state()

    match state:
        WebSocketPeer.STATE_OPEN:
            if not is_connected:
                is_connected = true
                connected.emit()
            while websocket.get_available_packet_count() > 0:
                _handle_packet(websocket.get_packet())
        WebSocketPeer.STATE_CLOSED:
            if is_connected:
                is_connected = false
                disconnected.emit()

func _handle_packet(data: PackedByteArray):
    var json_string = data.get_string_from_utf8()
    var json = JSON.parse_string(json_string)
    if json == null:
        return

    match json.get("type", ""):
        "STATE_SNAPSHOT":
            state_changed.emit(json.get("payload", {}))
        "PHASE_CHANGE":
            phase_changed.emit(json.get("phase", ""), json.get("endsAt", 0))
        "ERROR":
            error_received.emit(json.get("code", ""), json.get("message_es", ""))
        "OP_DELTA":
            _apply_delta(json.get("changes", []))

func send_message(type: String, payload: Dictionary = {}):
    if websocket == null or not is_connected:
        return
    var message = {"type": type, "payload": payload, "ts": Time.get_unix_time_from_system()}
    websocket.send_text(JSON.stringify(message))

func send_bid(property_id: String, amount: int, use_fame_token: bool = false):
    send_message("bid", {
        "propertyId": property_id,
        "amount": amount,
        "useFameToken": use_fame_token
    })

func send_action(action_type: String, params: Dictionary = {}):
    send_message("action", {
        "actionType": action_type,
        "params": params
    })
```

### 11.2.5 Scene Architecture

```
Match Scene Tree:
Match (Node2D)
├── CardManager (from card-framework)
├── Background
│   └── DistrictMap
├── GameArea
│   ├── MarketRow (Pile) ─── displays state.marketRow
│   ├── PlayerPortfolios (Control)
│   │   └── Portfolio_N (Hand) ─── displays owner.properties
│   └── EventDeck (Pile)
├── HUD (CanvasLayer)
│   ├── PhasePanel ─── switches based on state.phase
│   ├── CashDisplay
│   ├── PhaseTimer
│   ├── QuickChat
│   └── PlayerList
├── Overlays (CanvasLayer)
│   ├── VisualLedger
│   ├── DealPanel
│   ├── EventOverlay
│   └── AuctionOverlay
└── Audio
    ├── AmbientPlayer
    └── SFXPlayer
```

### 11.2.6 State Synchronization Pattern

```gdscript
# scenes/match/match.gd
extends Node2D

@onready var card_manager: CardManager = $CardManager
@onready var market_row: MarketRow = $GameArea/MarketRow
var property_cards: Dictionary = {}  # property_id -> PropertyCard

func _ready():
    ImperioClient.state_changed.connect(_on_state_changed)
    ImperioClient.phase_changed.connect(_on_phase_changed)

func _on_state_changed(state: Dictionary):
    # Update market row
    var market_properties = state.get("marketRow", [])
    for prop_id in market_properties:
        if prop_id not in property_cards:
            _create_property_card(prop_id, state.get("properties", {}).get(prop_id, {}))
        var card = property_cards[prop_id]
        if card.get_parent() != market_row.cards_node:
            market_row.add_card(card)

    # Update player portfolios
    for owner_id in state.get("owners", {}).keys():
        var owner = state.owners[owner_id]
        var portfolio = _get_portfolio(owner_id)
        for prop_id in owner.get("properties", []):
            var card = property_cards.get(prop_id)
            if card and card.get_parent() != portfolio.cards_node:
                portfolio.add_card(card)

func _create_property_card(prop_id: String, prop_state: Dictionary):
    var card_scene = preload("res://scenes/match/components/PropertyCard.tscn")
    var card = card_scene.instantiate()
    card.property_id = prop_id
    card.update_from_state(prop_state)
    property_cards[prop_id] = card
```

### 11.2.7 Mobile Input

Per original spec, use HTML DOM overlays for text input:
- Profile name entry
- Chat messages
- Room code entry

All other inputs use touch-friendly controls:
- Bid amounts: Preset buttons (+50K, +100K, +500K)
- Property selection: Tap on card
- Quick Chat: 12 preset buttons
- Deal creation: Template-based dropdowns

### 11.2.8 Export Configuration

Based on `alpapaydin` patterns:

```ini
# export_presets.cfg
[preset.0]
name="Web"
platform="Web"
runnable=true
custom_features=""
export_filter="all_resources"
include_filter="*.crt, *.pem"

[preset.0.options]
vram_texture_compression/for_desktop=true
vram_texture_compression/for_mobile=true
html/canvas_resize_policy=2
html/experimental_virtual_keyboard=false
progressive_web_app/enabled=false

[preset.1]
name="Linux Server"
platform="Linux/X11"
runnable=true
custom_features="dedicated_server"
export_filter="all_resources"
include_filter="*.crt, *.pem"
```

---

## Changes to Appendix O: Open Source Foundations

### REPLACE entire Appendix O with:

# APPENDIX O: OPEN SOURCE FOUNDATION DIRECTIVES

## O1: Committed Foundations

These are the **required** open source foundations for Imperio Inmobiliario.

### O1.1 Server
| Component | Repository | Version | License | Action |
|-----------|------------|---------|---------|--------|
| Framework | [colyseus/colyseus](https://github.com/colyseus/colyseus) | 0.15+ | MIT | Use official starter |
| Starter | `npm create colyseus-app@latest` | Latest | MIT | Initialize project |

### O1.2 Client
| Component | Repository | Version | License | Action |
|-----------|------------|---------|---------|--------|
| Engine | [godotengine/godot](https://github.com/godotengine/godot) | 4.5+ | MIT | Primary platform |
| Card UI | [chun92/card-framework](https://github.com/chun92/card-framework) | 1.3.1+ | MIT | Copy addon to project |
| WebSocket patterns | [alpapaydin/Godot4-Multiplayer-Survivor-IO-Game](https://github.com/alpapaydin/Godot4-Multiplayer-Survivor-IO-Game) | Reference | MIT | Adapt SSL/export code |

### O1.3 Database
| Component | Technology | Notes |
|-----------|------------|-------|
| Primary | PostgreSQL 15+ | Match snapshots, GD ledger |
| Cache | Redis (optional) | Rate limiting, sessions |

## O2: Excluded Repositories

The following were evaluated and **explicitly excluded**:

| Repository | Reason for Exclusion |
|------------|---------------------|
| sominator/colyseus-2d-card-game-templates | UNLICENSED, Godot 3.x, minimal value |
| db0/godot-card-game-framework | AGPL-3 license (copyleft incompatible) |
| gsioteam/godot-colyseus | Unclear Godot 4.x compatibility, low activity |

## O3: License Compliance

All committed foundations use MIT license. Compliance requirements:
1. Include MIT license text in distribution
2. Preserve copyright notices in source files
3. No additional restrictions

## O4: Reuse Matrix

| Imperio Component | Foundation Source | Integration Effort |
|-------------------|-------------------|-------------------|
| Card rendering | card-framework/card.gd | Low (extend class) |
| Card drag/drop | card-framework/draggable_object.gd | Low (use as-is) |
| Portfolio layout | card-framework/hand.gd | Low (extend class) |
| Market row | card-framework/pile.gd | Low (extend class) |
| Card manager | card-framework/card_manager.gd | Medium (add network events) |
| WebSocket connection | alpapaydin/Multihelper.gd | Medium (adapt for Colyseus) |
| SSL/TLS setup | alpapaydin/Constants.gd | Low (copy pattern) |
| Export presets | alpapaydin/export_presets.cfg | Low (copy and modify) |
| Colyseus room | Official examples | Medium (add game logic) |
| State schema | Custom | High (spec-specific) |
| Phase state machine | Custom | High (no existing match) |
| Economy formulas | Custom | High (spec-specific) |

## O5: Integration Steps

### Step 1: Initialize Server
```bash
npm create colyseus-app@latest imperio-server
cd imperio-server
npm install
```

### Step 2: Initialize Godot Client
```bash
# Create Godot 4.5+ project
# Download card-framework from Asset Library or GitHub
# Copy to addons/card-framework/
```

### Step 3: Configure Card Framework
1. Instance `card_manager.tscn` in Match scene
2. Create `PropertyCard` extending `Card`
3. Create `MarketRow` extending `Pile`
4. Create `Portfolio` extending `Hand`
5. Connect to `ImperioClient` signals

### Step 4: SSL for Production
1. Generate Let's Encrypt certificates
2. Place in `assets/certs/`
3. Set `Constants.USE_SSL = true`
4. Configure Colyseus server with certificates

---

## Implementation Plan Updates Required

After applying this amendment, the implementation plan (`plans/modular-implementation-plan.md`) must be expanded to include:

1. **Phase 7: Godot Client Foundation** (Week 13-14)
   - Initialize Godot 4.5+ project
   - Copy and configure card-framework addon
   - Create autoloads (Constants, ImperioClient, GameState)
   - Create base scenes (Lobby, Match, Results)

2. **Phase 8: Card UI Integration** (Week 15-16)
   - Extend card-framework classes (PropertyCard, MarketRow, Portfolio)
   - Implement state synchronization
   - Connect to Colyseus server

3. **Phase 9: Match UI** (Week 17-18)
   - Phase-specific panels (BidPanel, ActionPanel, DealPanel)
   - Visual Ledger animation
   - Quick Chat
   - Event overlays

4. **Phase 10: Polish & Export** (Week 19-20)
   - Mobile touch optimization
   - Audio integration
   - HTML5 export testing
   - SSL certificate deployment

---

## Changelog

| Section | Change |
|---------|--------|
| 1.5 | Replaced "Strategy" with "Commitments" — now directive, not evaluation |
| 11.2 | Expanded from 0.5 pages to 3 pages with full client architecture |
| 11.2.2 | NEW: Complete project structure |
| 11.2.3 | NEW: Card framework integration with code examples |
| 11.2.4 | NEW: WebSocket client with SSL (adapted from alpapaydin) |
| 11.2.5 | NEW: Scene architecture diagram |
| 11.2.6 | NEW: State synchronization pattern |
| 11.2.8 | NEW: Export configuration |
| Appendix O | Replaced evaluation matrix with directives |
| O1 | NEW: Committed foundations table |
| O2 | NEW: Excluded repositories with reasons |
| O4 | NEW: Reuse matrix |
| O5 | NEW: Integration steps |
