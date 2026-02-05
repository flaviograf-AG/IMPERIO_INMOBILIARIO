# Gaming Framework Research: Colyseus + Godot 4.x + PostgreSQL + Redis

Compiled for **IMPERIO INMOBILIARIO** — a multiplayer turn-based real estate tycoon game with an 11-phase state machine, 12 max owners, 20 max participants, and a Graf Dollar (GD) economy.

---

## 1. Colyseus Framework — Architecture Patterns for Turn-Based Multiplayer

### 1.1 Room Lifecycle Best Practices

Colyseus rooms follow a strict lifecycle. Every method supports async/await.

**Lifecycle Methods:**

| Method | When Called | Use For |
|--------|-----------|---------|
| `onCreate(options)` | Room created by matchmaker | Initialize state, set timers, load satellite data |
| `onJoin(client, options, auth)` | Client joins | Add player to state, assign role (Owner/Watcher) |
| `onLeave(client, consented)` | Client disconnects | Start reconnection window, flag inactive |
| `onDispose()` | Room destroyed (all left, or manual) | Persist match snapshot to PostgreSQL, cleanup |
| `onBeforeShutdown()` | Graceful shutdown signal | Warn players, save state, allow 5-min grace |

**Best Practices for ImperioRoom:**

```typescript
// ImperioRoom extends Room<ImperioState>
export class ImperioRoom extends Room<ImperioState> {
  maxClients = 20;

  async onCreate(options: RoomOptions) {
    // 1. Initialize state
    this.setState(new ImperioState());

    // 2. Load satellite data (properties.json, events.json, etc.)
    await this.loadSatelliteData();

    // 3. Set simulation interval for phase timer
    this.setSimulationInterval((dt) => this.gameLoop(dt), 1000);

    // 4. Register message handlers
    this.onMessage("bid", (client, data) => this.handleBid(client, data));
    this.onMessage("action", (client, data) => this.handleAction(client, data));
    this.onMessage("package", (client, data) => this.handlePackage(client, data));
    this.onMessage("quick_chat", (client, data) => this.handleQuickChat(client, data));

    // 5. Set patch rate (50ms = 20 updates/sec, fine for turn-based)
    this.setPatchRate(50);
  }
}
```

**Key Rules:**
- Keep `onCreate` lean. Delegate game logic to command objects or services.
- Never block the event loop in lifecycle methods. Use `await` for I/O.
- `onDispose` is your last chance to persist data. Always wrap in try/catch.
- `onBeforeShutdown` fires during deploy/restart. Use `this.clock.setTimeout` to delay disconnect and give players time to finish.

### 1.2 State Management with @colyseus/schema

**Schema Design Rules:**

1. **64 field limit per Schema class.** Use nested schemas liberally. ImperioState will easily exceed 64 fields without nesting.

2. **Schema = data only.** No game logic in schema classes. Use getters/setters only for computed properties that don't mutate state.

3. **Use correct collection types:**
   - `MapSchema<OwnerSchema>` for players (keyed by sessionId or ownerId)
   - `ArraySchema<PropertySchema>` for the property deck
   - `MapSchema` over `ArraySchema` when items are added/removed frequently (array shifts are expensive: removing index 0 from 20 items = 38 extra bytes serialized)

4. **Optimize number types.** Use `"uint8"` for upgrade levels (0-5), `"int32"` for GD cash amounts, `"uint16"` for round numbers. Default `"number"` uses 8 bytes (float64); explicit types save bandwidth.

5. **Use `@filter()` sparingly.** It consumes CPU per client per tick. For ImperioRoom, only filter truly secret data (e.g., hidden bid amounts before reveal). Prefer designing your schema so clients don't need hidden data.

6. **Non-synchronized fields are free.** Any field without `@type()` exists only on the server. Use this for internal bookkeeping (e.g., `roundSeed`, `serverSecret`).

**Example Schema Structure for Imperio:**

```typescript
class PropertySchema extends Schema {
  @type("string")  propertyId: string;
  @type("string")  name: string;
  @type("string")  district: string;
  @type("int32")   basePrice: number;      // GD base price
  @type("int32")   baseRent: number;       // GD base rent
  @type("uint8")   upgradeLevel: number;   // 0-5
  @type("string")  ownerId: string;        // "" if unowned
  @type("boolean") insured: boolean;
  @type("int32")   worthBonus: number;     // temp bonuses
  @type("int32")   rentBonus: number;      // temp bonuses

  // NOT synchronized -- server only
  riskModifier: number = 0;
}

class OwnerSchema extends Schema {
  @type("string")  ownerId: string;
  @type("string")  displayName: string;
  @type("int32")   cash: number;           // GD cash
  @type("int32")   debt: number;           // GD debt
  @type("int32")   wealth: number;         // computed per round
  @type("uint8")   fame: number;           // 0-100
  @type("string")  dynastyId: string;
  @type("boolean") isConnected: boolean;
  @type("boolean") isBot: boolean;

  // NOT synchronized
  disconnectedAtRound: number = -1;
}

class ImperioState extends Schema {
  @type("string")           phase: string;          // current phase name
  @type("uint8")            roundNumber: number;
  @type("uint8")            totalRounds: number;
  @type({ map: OwnerSchema })  owners = new MapSchema<OwnerSchema>();
  @type([ PropertySchema ]) properties = new ArraySchema<PropertySchema>();
  @type("int32")            phaseTimer: number;     // ms remaining
  @type("string")           currentSignal: string;  // market signal
  @type("string")           headline: string;       // press headline

  // NOT synchronized
  matchSeed: string;
  serverSecret: string;
}
```

**Version Compatibility:** Both server and client MUST share identical schema definitions. Use `schema-codegen` to generate client-side code. Add new fields only at the end of classes. Mark removed fields as `@deprecated()` -- never delete them.

### 1.3 Clock/Timer Management for Phase-Based Games

The `this.clock` (`ClockTimer`) is the right tool for phase timing. It operates independently of state synchronization.

**Phase Timer Pattern:**

```typescript
private phaseDelayed: Delayed | null = null;

private startPhase(phaseName: string, durationMs: number) {
  this.state.phase = phaseName;
  this.state.phaseTimer = durationMs;

  // Clear any existing timer
  if (this.phaseDelayed) this.phaseDelayed.clear();

  // Set phase timeout
  this.phaseDelayed = this.clock.setTimeout(() => {
    this.onPhaseTimeout(phaseName);
  }, durationMs);

  // Update client-visible timer every second
  this.clock.setInterval(() => {
    this.state.phaseTimer = Math.max(0, this.state.phaseTimer - 1000);
  }, 1000);
}

private onPhaseTimeout(phaseName: string) {
  // Server proceeds regardless of client response
  // No bid = no bid, no action = Pass, no vote = not counted
  this.advanceToNextPhase();
}
```

**Key Clock Methods:**
- `clock.setTimeout(fn, ms)` -- returns `Delayed`. Use for phase deadlines.
- `clock.setInterval(fn, ms)` -- returns `Delayed`. Use for countdown updates.
- `delayed.pause()` / `delayed.resume()` -- use for pause functionality.
- `delayed.clear()` -- cancel timer.
- `clock.clear()` -- cancel all timers. Auto-called on room dispose.

**Visual Ledger Wait Pattern:**
The spec requires a 4.0s Visual Ledger animation where the server waits. Use `clock.setTimeout` to advance after exactly 4000ms. Do NOT wait for client confirmation:

```typescript
// INCOME_CALC -> VISUAL_LEDGER -> ACTION
this.startPhase("VISUAL_LEDGER", 4000);
// After 4s, automatically advance to ACTION phase
```

### 1.4 Reconnection Handling Patterns

Colyseus supports reconnection via `allowReconnection()` in `onLeave()`.

**Pattern for Turn-Based Game with Bot Takeover:**

```typescript
async onLeave(client: Client, consented: boolean) {
  const owner = this.state.owners.get(client.sessionId);
  if (!owner) return; // Watcher left, no special handling

  owner.isConnected = false;
  owner.disconnectedAtRound = this.state.roundNumber;

  try {
    if (consented) {
      // Player explicitly quit -- immediate bot takeover
      throw new Error("consented leave");
    }

    // Allow 60s reconnection window (per spec)
    const reconnection = this.allowReconnection(client, 60);

    // Reject reconnection if player missed too many rounds
    const checkInterval = this.clock.setInterval(() => {
      if (this.state.roundNumber - owner.disconnectedAtRound > 2) {
        reconnection.reject();
        checkInterval.clear();
      }
    }, 1000);

    await reconnection;

    // Player reconnected successfully
    owner.isConnected = true;
    owner.disconnectedAtRound = -1;
    checkInterval.clear();

  } catch (e) {
    // Reconnection failed or rejected -- activate bot (A8 heuristics)
    owner.isBot = true;
    owner.isConnected = false;
    // Do NOT remove from state -- bot plays on their behalf
  }
}
```

**Rate Limiting Reconnection:** The spec requires max 5 reconnect attempts/min. Track this server-side using a simple counter per sessionId, resetting every 60s.

### 1.5 Matchmaking Strategies

Colyseus provides several matchmaking mechanisms.

**Room Filtering with `filterBy()`:**

```typescript
// Server definition
gameServer
  .define("imperio_quick", ImperioRoom, { mode: "quick", rounds: 7 })
  .filterBy(["mode"]);

gameServer
  .define("imperio_standard", ImperioRoom, { mode: "standard", rounds: 10 })
  .filterBy(["mode"]);

gameServer
  .define("imperio_long", ImperioRoom, { mode: "long", rounds: 14 })
  .filterBy(["mode"]);
```

**Ranked Matchmaking Queue (ELO/MMR):**

Colyseus provides `colyseus-ranked-matchmaking` as a reference implementation. The pattern:

1. Player joins a `RankedQueueRoom` with their rank (e.g., GD lifetime wallet as proxy).
2. Every `cycleTickInterval` (default 1s), the server evaluates compatible groups.
3. Once a compatible group is found, a game room is created with reserved seats.
4. Clients connect via `.consumeSeatReservation()` and disconnect from the queue.

```typescript
// Extending the ranked queue for Imperio
class ImperioQueueRoom extends RankedQueueRoom {
  async onAuth(client: Client, options: any) {
    const token = await verifyToken(options.token);
    return { userId: token.userId, rank: token.lifetimeGD };
  }

  onGroupReady(group: MatchGroup) {
    // Fill remaining slots with bots
    const roomOptions = {
      numBots: this.maxPlayers - group.clients.length,
      mode: "standard",
    };
    return { roomName: "imperio_standard", roomOptions };
  }
}
```

**Casual vs. Ranked:** Use `filterBy(["mode", "ranked"])` to separate casual (open join) from ranked (queue-based) rooms.

**Lobby Room:** Use the built-in `LobbyRoom` with `enableRealtimeListing()` to show available rooms to clients. It auto-updates when rooms are created/joined/left/disposed.

### 1.6 Scalability: Horizontal Scaling

**Core Architecture:**
- Each Room lives in one Colyseus process.
- Multiple processes share state via **Redis Presence** and **Redis Driver**.
- A load balancer distributes incoming connections across processes.

**Setup:**

```typescript
import { Server } from "colyseus";
import { RedisPresence } from "@colyseus/redis-presence";
import { RedisDriver } from "@colyseus/redis-driver";

const gameServer = new Server({
  presence: new RedisPresence({ host: "127.0.0.1", port: 6379 }),
  driver: new RedisDriver({ host: "127.0.0.1", port: 6379 }),
  // Each process listens on a different port
  publicAddress: `game.imperio.io:${process.env.PORT}`,
});
```

**Two-Step Seat Reservation:**
1. Any Colyseus process handles the join request and reserves a seat (via Redis).
2. The client receives the address of the specific process hosting that room.
3. The client connects directly to that process via WebSocket.

**Process Selection:** By default, Colyseus creates rooms on the process with the fewest rooms. Customize via `selectProcessIdToCreateRoom`:

```typescript
gameServer.define("imperio", ImperioRoom)
  .selectProcessIdToCreateRoom((processes) => {
    // Prefer process with lowest CCU (concurrent users)
    return processes.sort((a, b) => a.ccu - b.ccu)[0].processId;
  });
```

**PM2 for Process Management:** Run multiple instances on ports 2567-2570+. Use `ecosystem.config.js` with `instances: 4`.

**Capacity Planning:** A single Colyseus process on a modest server can handle around 3,000 concurrent WebSocket connections. For a turn-based game with rooms of 20 players max, that is around 150 concurrent rooms per process. With 4 processes: around 600 rooms, around 12,000 players.

### 1.7 Common Pitfalls

| Pitfall | Impact | Mitigation |
|---------|--------|------------|
| **State size bloat** | Increased bandwidth, slow sync | Only `@type()` what clients need. Use specific number types. Keep arrays small. |
| **Array shift/unshift operations** | +2 bytes per element per operation | Use `MapSchema` for collections that change frequently. Avoid `shift()`. |
| **Game logic in Schema classes** | Hard to test, hard to maintain | Use Command pattern. Schemas = data only. |
| **Blocking the event loop** | All rooms on that process freeze | Use `async/await` for I/O. Never `JSON.parse` large payloads synchronously. |
| **Not handling `onDispose`** | Lost match data | Always persist match snapshots in `onDispose`. |
| **Client/server schema mismatch** | Deserialization crashes | Use `schema-codegen`. Add fields at the end. Never delete fields. |
| **Large initial state** | Slow first sync on join | Lazy-load non-essential data. Send satellite data via messages, not state. |
| **Forgetting `clock.clear()`** | Timer leaks across phases | Clear timers when advancing phases. Auto-cleared on dispose. |
| **State persistence gap** | Crash = lost game | Colyseus keeps state in-memory only. Periodic snapshots to Redis/PostgreSQL recommended. |

### 1.8 Version Compatibility Between Client/Server Schemas

- **Adding fields:** Always add new `@type()` fields at the END of the class. Both old and new clients will work.
- **Removing fields:** Mark as `@deprecated()` -- never physically delete. Old clients ignore deprecated fields. New clients skip them.
- **Changing field types:** NEVER change a field's type. Add a new field with the new type and deprecate the old one.
- **Schema codegen:** Run `npx schema-codegen` after every schema change to regenerate client-side decoders (C#, Lua, GDScript, etc.).
- **Collection item types:** All items in an `ArraySchema` or `MapSchema` must be the same type. Mixing types causes serialization failures.

---

## 2. Godot 4.x for Multiplayer -- Client-Side Patterns

### 2.1 WebSocket Client for Colyseus Connection

**Option A: godot-colyseus SDK (GDScript)**

A community SDK (`gsioteam/godot-colyseus`) mirrors the `colyseus.js` API. Originally written for Godot 3, it needs adaptation for Godot 4's `await` syntax.

```gdscript
# Godot 4 adapted pattern
var client = Colyseus.Client.new("ws://game.imperio.io:2567")
var room = await client.join_or_create("imperio_standard", { "mode": "standard" })

room.on_state_change.connect(_on_state_changed)
room.on_message("PHASE_CHANGE", _on_phase_change)
room.on_message("QUICK_CHAT_MSG", _on_quick_chat)
room.on_error.connect(_on_error)
room.on_leave.connect(_on_leave)
```

**Option B: Raw WebSocketPeer (Manual Protocol)**

For tighter control or if the SDK doesn't support Godot 4 well:

```gdscript
extends Node

var ws = WebSocketPeer.new()
var connected := false

func _ready():
    var err = ws.connect_to_url("ws://game.imperio.io:2567/imperio_standard")
    if err != OK:
        push_error("Connection failed: %s" % err)

func _process(_delta):
    ws.poll()
    var state = ws.get_ready_state()
    match state:
        WebSocketPeer.STATE_OPEN:
            if not connected:
                connected = true
                _on_connected()
            while ws.get_available_packet_count() > 0:
                var packet = ws.get_packet()
                _handle_message(packet)
        WebSocketPeer.STATE_CLOSED:
            if connected:
                connected = false
                _on_disconnected()

func send_message(type: String, payload: Dictionary):
    var envelope = {
        "type": type,
        "payload": payload,
        "ts": Time.get_unix_time_from_system(),
        "idempotencyKey": _generate_uuid()
    }
    ws.send_text(JSON.stringify(envelope))
```

**Option C: GodotJS + Official colyseus.js**

If using GodotExplorer/ECMAScript module, you can use the official JavaScript Colyseus client directly via npm. This gives full feature parity but adds a JS runtime dependency.

**Recommendation for Imperio:** Start with Option B (raw WebSocket) for maximum control over the protocol and compatibility with Godot 4.x web export. The Colyseus wire protocol is well-documented and the overhead of manual implementation is low for a turn-based game.

### 2.2 Scene Management for Game Phases

**Phase-to-Scene Mapping:**

```
LOBBY          -> LobbyScene (player list, mode selection, ready)
MATCH_INIT     -> TransitionScene (loading, brief animation)
REVEAL         -> RevealScene (property card flip animation)
BID            -> BidScene (auction UI, timer)
INCOME_CALC    -> IncomeScene (overlay on main board)
VISUAL_LEDGER  -> LedgerScene (animated income/expense waterfall)
ACTION         -> ActionScene (major + minor action selection)
MOOD           -> MoodScene (8 mood options)
DEALS          -> DealsScene (7 deal templates)
EVENTS         -> EventScene (targeted event card, impact display)
PRESS          -> PressScene (headline ticker)
AWARDS/END     -> ResultsScene (final rankings, GD awarded)
```

**Scene Transition Pattern with signals:**

```gdscript
# PhaseManager.gd (Autoload Singleton)
signal phase_changed(old_phase: String, new_phase: String)
signal phase_timer_updated(remaining_ms: int)

var current_phase: String = "LOBBY"
var phase_scenes: Dictionary = {
    "LOBBY": preload("res://scenes/phases/LobbyScene.tscn"),
    "REVEAL": preload("res://scenes/phases/RevealScene.tscn"),
    "BID": preload("res://scenes/phases/BidScene.tscn"),
    # ... etc
}

func transition_to(new_phase: String):
    var old = current_phase
    current_phase = new_phase

    # Fade out current scene
    var tween = create_tween()
    tween.tween_property($CurrentScene, "modulate:a", 0.0, 0.3)
    await tween.finished

    # Swap scene
    var new_scene = phase_scenes[new_phase].instantiate()
    $SceneContainer.add_child(new_scene)
    $CurrentScene.queue_free()

    # Fade in
    new_scene.modulate.a = 0.0
    tween = create_tween()
    tween.tween_property(new_scene, "modulate:a", 1.0, 0.3)

    phase_changed.emit(old, new_phase)
```

**Best Practice:** Do NOT use `get_tree().change_scene()` for phase transitions. Use a persistent main scene with a `SubViewportContainer` or `CanvasLayer` for swapping phase content. The main board/map should remain visible throughout most phases.

### 2.3 UI Patterns for Real-Time Data

**Property Cards:**
Use `PanelContainer` with a `VBoxContainer`. Bind to state changes:

```gdscript
func _on_property_state_changed(property_id: String, changes: Dictionary):
    var card = property_cards[property_id]
    if "upgradeLevel" in changes:
        card.update_stars(changes["upgradeLevel"])
    if "ownerId" in changes:
        card.update_owner_badge(changes["ownerId"])
    if "rentBonus" in changes:
        card.animate_rent_change(changes["rentBonus"])
```

**Leaderboard:**
Use a `VBoxContainer` with dynamically sorted children. Re-sort on wealth changes:

```gdscript
func update_leaderboard(owners: Array):
    owners.sort_custom(func(a, b): return a.wealth > b.wealth)
    for i in range(owners.size()):
        var row = leaderboard_rows[owners[i].ownerId]
        # Animate position change
        var target_y = i * ROW_HEIGHT
        var tween = create_tween()
        tween.tween_property(row, "position:y", target_y, 0.4) \
            .set_ease(Tween.EASE_OUT) \
            .set_trans(Tween.TRANS_CUBIC)
```

**Phase Timer:**
Use a `ProgressBar` or custom shader-based radial timer:

```gdscript
func _process(delta):
    if phase_timer_max > 0:
        timer_bar.value = (phase_timer_remaining / phase_timer_max) * 100
        timer_label.text = "%d" % ceil(phase_timer_remaining / 1000.0)

        # Urgency effect when < 5s
        if phase_timer_remaining < 5000:
            timer_bar.modulate = Color(1, 0.3, 0.3)  # Red pulse
```

### 2.4 WebAssembly Export Limitations and Workarounds

| Limitation | Workaround |
|-----------|------------|
| **Large WASM binary (around 33MB default)** | Build custom export templates stripping unused modules (3D renderer, physics, etc.). A 2D-focused build can be around 2-5MB. |
| **SharedArrayBuffer requirement (pre-4.3)** | Use Godot 4.3+ with single-threaded export option (disable Thread Support in web export settings). |
| **Mobile browser performance** | Strip 3D renderer entirely. Use 2D sprites for the board. Minimize shader complexity. Test on low-end Android devices. |
| **C# not supported for web** | Use GDScript only. No C# support for WebAssembly export. |
| **No native file system** | Use IndexedDB via JavaScript interop for local caching. |
| **Audio autoplay blocked by browsers** | Require user interaction (tap/click) before playing audio. Use `AudioServer` after first input event. |
| **No clipboard access** | Use JavaScript interop (`JavaScriptBridge.eval()`) for copy/paste. |
| **Debugging** | Use Chrome's "C/C++ DevTools Support" extension for WASM breakpoints. GDScript print statements output to browser console. |

**Critical for Imperio:** Since the game targets mobile web, the WASM binary size is the number one concern. Recommended approach:
1. Disable 3D rendering, physics engines, and unused modules at compile time.
2. Use Godot 4.3+ for single-threaded export (no COOP/COEP headers needed).
3. Use progressive loading: show the lobby immediately, load game assets in background.
4. Compress with Brotli on the web server (nginx `brotli_static on`).

### 2.5 Mobile Input Handling

**Touch Events:**
Enable `emulate_touch_from_mouse` in Project Settings for desktop testing. Use `InputEventScreenTouch` and `InputEventScreenDrag` for mobile.

**Gesture Library (GDTIM):**
The Godot Touch Input Manager plugin adds swipe, pinch, long-press, and multi-finger gesture detection. Install from the Asset Library.

**Card Game Gestures:**

```gdscript
# Swipe to browse properties
func _on_swipe(direction: Vector2):
    if direction.x > 0:
        show_next_property()
    elif direction.x < 0:
        show_previous_property()

# Long press for property details
func _on_long_press(position: Vector2):
    var property = get_property_at(position)
    if property:
        show_property_details_popup(property)

# Tap to select action
func _on_tap(position: Vector2):
    var action = get_action_at(position)
    if action:
        select_action(action)
```

**HTML DOM Overlays for Text Input (per spec):**
Godot's virtual keyboard on mobile web is unreliable. Use JavaScript interop to create HTML `<input>` elements overlaid on the Godot canvas:

```gdscript
func show_text_input(placeholder: String, callback: Callable):
    JavaScriptBridge.eval("""
        var input = document.createElement('input');
        input.type = 'text';
        input.placeholder = '%s';
        input.style.position = 'absolute';
        input.style.top = '50%%';
        input.style.left = '10%%';
        input.style.width = '80%%';
        input.style.fontSize = '18px';
        input.style.zIndex = '9999';
        document.body.appendChild(input);
        input.focus();
    """ % placeholder)
```

### 2.6 Animation Systems for Board-Game-Like Visual Feedback

**AnimationPlayer for Preset Sequences:**
- Property card flip (REVEAL phase): Use `AnimationPlayer` with rotation + scale keyframes
- Visual Ledger waterfall: Tween-based cascading numbers
- Event card dramatic entrance: Scale from 0 + shake effect
- Bid counter animation: Rolling number effect

**Tween API for Dynamic Animations:**

```gdscript
# GD cash change animation (income/expense)
func animate_cash_change(amount: int, is_positive: bool):
    var label = cash_change_label.duplicate()
    add_child(label)
    label.text = "%s GD %s" % ["+" if is_positive else "-", format_gd(abs(amount))]
    label.modulate = Color.GREEN if is_positive else Color.RED

    var tween = create_tween()
    tween.set_parallel(true)
    tween.tween_property(label, "position:y", label.position.y - 60, 1.0)
    tween.tween_property(label, "modulate:a", 0.0, 1.0).set_delay(0.5)
    tween.finished.connect(label.queue_free)
```

**Shader Effects:**
- Gold shimmer on high-value properties (Tycoon Luxury style)
- Pulse glow on actionable elements during ACTION phase
- Screen shake on negative events
- Particle effects for upgrades and purchases

### 2.7 Audio Management for Dynamic Soundscapes

**Layered Audio Pattern:**

```gdscript
# AudioManager.gd (Autoload Singleton)
var layers: Dictionary = {
    "ambient": $AmbientPlayer,    # Lima city ambiance -- always playing
    "music": $MusicPlayer,        # Background music -- mood-dependent
    "tension": $TensionPlayer,    # Layered in during bid phase
    "event": $EventPlayer,        # Stinger for events
}

func on_phase_changed(phase: String):
    match phase:
        "BID":
            fade_in(layers["tension"], 0.5)
        "EVENTS":
            play_stinger("event_dramatic")
            fade_out(layers["tension"], 0.3)
        "VISUAL_LEDGER":
            play_sfx("coins_cascade")
        _:
            fade_out(layers["tension"], 0.5)

func fade_in(player: AudioStreamPlayer, duration: float):
    player.volume_db = -80
    player.play()
    var tween = create_tween()
    tween.tween_property(player, "volume_db", 0, duration)
```

**Audio Bus Structure:**
- Master bus
- Music bus (with compressor)
- SFX bus (higher priority)
- Ambient bus (low volume, looping)
- UI bus (clicks, hovers -- short, crisp)

---

## 3. PostgreSQL for Game Data -- Persistence Patterns

### 3.1 Match History Storage: JSONB Snapshots vs. Normalized Tables

**Recommended Hybrid Approach for Imperio:**

Use **normalized tables** for queryable data (leaderboards, player stats, match metadata) and **JSONB** for full match state snapshots (replay, debugging, audit).

```sql
-- Normalized: fast queries, indexes, aggregations
CREATE TABLE matches (
    match_id    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    mode        TEXT NOT NULL CHECK (mode IN ('quick', 'standard', 'long')),
    rounds      SMALLINT NOT NULL,
    started_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    ended_at    TIMESTAMPTZ,
    winner_id   UUID REFERENCES players(player_id),
    status      TEXT NOT NULL DEFAULT 'active'
);

CREATE TABLE match_players (
    match_id    UUID REFERENCES matches(match_id),
    player_id   UUID REFERENCES players(player_id),
    role        TEXT NOT NULL CHECK (role IN ('owner', 'watcher')),
    final_wealth BIGINT,            -- GD at match end
    final_rank  SMALLINT,
    dynasty_id  TEXT,
    PRIMARY KEY (match_id, player_id)
);

-- JSONB: full match replay data
CREATE TABLE match_snapshots (
    snapshot_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    match_id    UUID REFERENCES matches(match_id),
    round       SMALLINT NOT NULL,
    phase       TEXT NOT NULL,
    state_data  JSONB NOT NULL,     -- full ImperioState serialized
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- GIN index for querying inside JSONB
CREATE INDEX idx_snapshots_state ON match_snapshots USING GIN (state_data);

-- Partial index: only index active matches for recent queries
CREATE INDEX idx_matches_active ON matches (started_at DESC)
    WHERE status = 'active';
```

**When to use JSONB:**
- Match state snapshots (one per round per match = 10-14 rows per match)
- Event history within a round
- Flexible metadata that changes with spec versions

**When to use normalized columns:**
- Player lifetime stats (queryable, aggregatable)
- Leaderboard rankings
- Graf Dollar ledger (critical financial data needs relational integrity)

### 3.2 Leaderboard Queries at Scale

**Materialized View Pattern:**

```sql
-- Materialized view for global leaderboard
CREATE MATERIALIZED VIEW leaderboard AS
SELECT
    p.player_id,
    p.display_name,
    p.dynasty_id,
    COALESCE(bc.balance, 0) AS lifetime_gd,
    COUNT(DISTINCT mp.match_id) AS matches_played,
    COUNT(DISTINCT mp.match_id) FILTER (WHERE mp.final_rank = 1) AS wins,
    RANK() OVER (ORDER BY COALESCE(bc.balance, 0) DESC) AS global_rank
FROM players p
LEFT JOIN balance_cache bc ON bc.user_id = p.player_id
LEFT JOIN match_players mp ON mp.player_id = p.player_id
GROUP BY p.player_id, p.display_name, p.dynasty_id, bc.balance
WITH DATA;

-- Required unique index for CONCURRENTLY refresh
CREATE UNIQUE INDEX idx_leaderboard_player ON leaderboard (player_id);

-- Refresh without locking (PostgreSQL 9.4+)
-- Schedule via pg_cron every 5-15 minutes
SELECT cron.schedule('refresh_leaderboard', '*/5 * * * *',
    'REFRESH MATERIALIZED VIEW CONCURRENTLY leaderboard');
```

**Dynasty Leaderboard:**

```sql
CREATE MATERIALIZED VIEW dynasty_leaderboard AS
SELECT
    dynasty_id,
    SUM(COALESCE(bc.balance, 0)) AS total_gd,
    COUNT(DISTINCT p.player_id) AS member_count,
    RANK() OVER (ORDER BY SUM(COALESCE(bc.balance, 0)) DESC) AS dynasty_rank
FROM players p
LEFT JOIN balance_cache bc ON bc.user_id = p.player_id
WHERE p.dynasty_id IS NOT NULL
GROUP BY dynasty_id
WITH DATA;
```

**Indexing Strategies:**
- B-tree on `balance_cache.balance DESC` for top-N queries
- Composite index on `(dynasty_id, balance)` for dynasty rankings
- GIN index on JSONB columns only when you query inside them
- Partial indexes for active/recent data to keep index sizes small

### 3.3 Double-Entry Ledger for Graf Dollar

The `graf_dollar.txt` spec defines the ledger schema. Key implementation details:

**Tables:** `ledger_tx`, `ledger_entry`, `balance_cache`, `loan_contract` (Phase 3)

**Critical Pattern -- Atomic Award with Balance Cache:**

```sql
-- Award GD to match winner (server-only operation)
CREATE OR REPLACE FUNCTION award_gd(
    p_match_id UUID,
    p_user_id UUID,
    p_amount BIGINT,
    p_reward_type TEXT
) RETURNS TABLE(tx_id UUID, new_balance BIGINT) AS $$
DECLARE
    v_tx_id UUID := gen_random_uuid();
    v_idem_key TEXT := 'award:' || p_match_id || ':' || p_user_id || ':' || p_reward_type;
    v_existing UUID;
    v_balance BIGINT;
BEGIN
    -- Idempotency check
    SELECT lt.tx_id INTO v_existing
    FROM ledger_tx lt WHERE lt.idempotency_key = v_idem_key;

    IF v_existing IS NOT NULL THEN
        -- Return existing result
        SELECT bc.balance INTO v_balance FROM balance_cache bc WHERE bc.user_id = p_user_id;
        RETURN QUERY SELECT v_existing, COALESCE(v_balance, 0::BIGINT);
        RETURN;
    END IF;

    -- Insert transaction
    INSERT INTO ledger_tx (tx_id, tx_type, created_by, idempotency_key, metadata)
    VALUES (v_tx_id, 'award', 'server', v_idem_key,
            jsonb_build_object('match_id', p_match_id, 'reward_type', p_reward_type));

    -- Double-entry: debit system, credit player
    INSERT INTO ledger_entry (entry_id, tx_id, user_id, amount) VALUES
        (gen_random_uuid(), v_tx_id, 'SYSTEM_USER_ID', -p_amount),
        (gen_random_uuid(), v_tx_id, p_user_id, p_amount);

    -- Update balance cache atomically
    INSERT INTO balance_cache (user_id, balance, updated_at)
    VALUES (p_user_id, p_amount, now())
    ON CONFLICT (user_id) DO UPDATE SET
        balance = balance_cache.balance + p_amount,
        updated_at = now();

    SELECT bc.balance INTO v_balance FROM balance_cache bc WHERE bc.user_id = p_user_id;
    RETURN QUERY SELECT v_tx_id, v_balance;
END;
$$ LANGUAGE plpgsql;
```

**Concurrency:** Use `SELECT ... FOR UPDATE` on `balance_cache` rows before checking balance for debit operations. This prevents double-spend race conditions.

**Isolation Level:** Use `REPEATABLE READ` for the ledger transactions. `SERIALIZABLE` is safer but has higher contention; `REPEATABLE READ` catches most conflicts via snapshot isolation.

**Audit Job:**

```sql
-- Periodic audit: verify all transactions sum to zero
SELECT tx_id, SUM(amount) AS entry_sum
FROM ledger_entry
GROUP BY tx_id
HAVING SUM(amount) != 0;
-- This query MUST return 0 rows. Any results indicate a bug.

-- Verify cache matches ledger
SELECT bc.user_id, bc.balance AS cached, COALESCE(le.actual, 0) AS actual
FROM balance_cache bc
LEFT JOIN (
    SELECT user_id, SUM(amount) AS actual
    FROM ledger_entry
    GROUP BY user_id
) le ON le.user_id = bc.user_id
WHERE bc.balance != COALESCE(le.actual, 0);
-- This query MUST also return 0 rows.
```

### 3.4 Concurrent Write Patterns for Real-Time Games

**Advisory Locks for Match State:**

```sql
-- Use advisory lock per match to prevent concurrent state writes
SELECT pg_advisory_xact_lock(hashtext(match_id::text));
-- ... perform match state updates ...
-- Lock automatically released on commit/rollback
```

**Batching Snapshot Writes:**
Instead of writing to PostgreSQL on every phase change, batch writes per round:

```typescript
// Server-side: collect all phase data, write once at SNAPSHOT phase
async onSnapshotPhase() {
  const snapshot = this.state.toJSON();
  await db.query(
    `INSERT INTO match_snapshots (match_id, round, phase, state_data)
     VALUES ($1, $2, 'SNAPSHOT', $3)`,
    [this.matchId, this.state.roundNumber, JSON.stringify(snapshot)]
  );
}
```

**Connection Pooling:** Use `pg-pool` with `max: 20` connections per Colyseus process. For 4 processes, that is 80 total connections -- well within PostgreSQL's default `max_connections = 100`.

### 3.5 Data Retention and Archival Strategies

**Partitioning by Month:**

```sql
-- Partition match_snapshots by month
CREATE TABLE match_snapshots (
    snapshot_id UUID NOT NULL DEFAULT gen_random_uuid(),
    match_id    UUID NOT NULL,
    round       SMALLINT NOT NULL,
    phase       TEXT NOT NULL,
    state_data  JSONB NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
) PARTITION BY RANGE (created_at);

-- Create monthly partitions
CREATE TABLE match_snapshots_2026_01 PARTITION OF match_snapshots
    FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

CREATE TABLE match_snapshots_2026_02 PARTITION OF match_snapshots
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');
```

**Retention Policy with pg_partman:**
- Hot data (last 3 months): SSD storage, full indexes
- Warm data (3-12 months): Detach partitions, compress, move to cheaper tablespace
- Cold data (12+ months): Dump to S3/archive storage, drop partitions

**Automated Cleanup:**

```sql
-- Schedule monthly partition management via pg_cron
SELECT cron.schedule('archive_old_snapshots', '0 2 1 * *',
    $$ALTER TABLE match_snapshots DETACH PARTITION match_snapshots_old CONCURRENTLY$$);
```

**Ledger Data:** NEVER delete or archive ledger data. The Graf Dollar ledger is append-only by design. If space becomes a concern, use tablespace tiering (SSD for recent, HDD for old) but keep it queryable.

---

## 4. Redis for Real-Time Gaming -- Caching and Session Patterns

### 4.1 Rate Limiting Implementations

**Sliding Window Rate Limiter:**

```typescript
// Rate limit: 5 reconnect attempts per minute per client
async function checkReconnectRateLimit(clientId: string): Promise<boolean> {
  const key = `ratelimit:reconnect:${clientId}`;
  const now = Date.now();
  const windowMs = 60_000; // 1 minute

  const pipeline = redis.pipeline();
  // Remove entries outside the window
  pipeline.zremrangebyscore(key, 0, now - windowMs);
  // Add current attempt
  pipeline.zadd(key, now, `${now}-${Math.random()}`);
  // Count entries in window
  pipeline.zcard(key);
  // Set expiry on the key itself
  pipeline.expire(key, 120);

  const results = await pipeline.exec();
  const count = results[2][1] as number;

  return count <= 5; // Allow if 5 or fewer attempts
}
```

**Per-Round Action Rate Limits:**

```typescript
// 1 bid per round, 1 action per round, 1 vote per round
async function checkRoundAction(
  matchId: string, playerId: string,
  roundNum: number, actionType: string
): Promise<boolean> {
  const key = `action:${matchId}:${roundNum}:${playerId}:${actionType}`;
  // SETNX returns 1 if set (first time), 0 if already exists
  const result = await redis.set(key, "1", "EX", 300, "NX");
  return result === "OK";
}
```

### 4.2 Session Management for Reconnection

```typescript
// Store session data on connect
async function storeSession(sessionId: string, data: SessionData) {
  await redis.set(
    `session:${sessionId}`,
    JSON.stringify(data),
    "EX",
    3600  // 1 hour TTL
  );
}

// Retrieve on reconnect
async function getSession(sessionId: string): Promise<SessionData | null> {
  const raw = await redis.get(`session:${sessionId}`);
  return raw ? JSON.parse(raw) : null;
}

// Store reconnection token for client
async function storeReconnectionToken(
  sessionId: string, roomId: string, reconnectionToken: string
) {
  await redis.set(
    `reconnect:${sessionId}`,
    JSON.stringify({ roomId, reconnectionToken }),
    "EX",
    60  // 60s reconnection window per spec
  );
}
```

### 4.3 Pub/Sub for Cross-Room Events

```typescript
// Cross-room event broadcasting
// Example: notify lobby about room state changes
const subscriber = redis.duplicate();
await subscriber.subscribe("imperio:rooms:updates");

subscriber.on("message", (channel, message) => {
  const event = JSON.parse(message);
  // Update lobby room listing
  lobbyRoom.broadcast("room_update", event);
});

// From a game room, publish updates
function publishRoomUpdate(roomId: string, status: string, playerCount: number) {
  redis.publish("imperio:rooms:updates", JSON.stringify({
    roomId,
    status,
    playerCount,
    updatedAt: Date.now(),
  }));
}
```

**Pattern-Based Subscriptions:**

```typescript
// Listen to all match events across rooms
await subscriber.psubscribe("imperio:match:*");
subscriber.on("pmessage", (pattern, channel, message) => {
  // channel = "imperio:match:abc123:event"
  const [, , matchId, eventType] = channel.split(":");
  // Handle cross-room analytics, spectator updates, etc.
});
```

**Caveat:** Redis Pub/Sub is fire-and-forget. If a subscriber is down, it misses messages. For critical events (match results, GD awards), use PostgreSQL transactions, NOT Redis Pub/Sub.

### 4.4 Sorted Sets for Real-Time Leaderboards

```typescript
// Update player's lifetime GD in leaderboard
async function updateLeaderboard(playerId: string, newGD: number) {
  await redis.zadd("leaderboard:global", newGD, playerId);
}

// Get top 100 players
async function getTopPlayers(count: number = 100): Promise<LeaderboardEntry[]> {
  const results = await redis.zrevrange(
    "leaderboard:global", 0, count - 1, "WITHSCORES"
  );
  const entries: LeaderboardEntry[] = [];
  for (let i = 0; i < results.length; i += 2) {
    entries.push({
      playerId: results[i],
      lifetimeGD: parseInt(results[i + 1]),
      rank: (i / 2) + 1,
    });
  }
  return entries;
}

// Get a specific player's rank
async function getPlayerRank(playerId: string): Promise<number | null> {
  const rank = await redis.zrevrank("leaderboard:global", playerId);
  return rank !== null ? rank + 1 : null;
}

// Per-dynasty leaderboard
async function updateDynastyLeaderboard(dynastyId: string, totalGD: number) {
  await redis.zadd("leaderboard:dynasty", totalGD, dynastyId);
}

// Seasonal leaderboard (separate key per season)
async function updateSeasonalLeaderboard(
  season: string, playerId: string, gd: number
) {
  await redis.zadd(`leaderboard:season:${season}`, gd, playerId);
}
```

**Leaderboard with Player Details:**
Store player details in a hash alongside the sorted set:

```typescript
// Store player info
await redis.hset(`player:${playerId}`, {
  displayName: "Carlos",
  dynastyId: "lima_legends",
  avatar: "tycoon_03",
});

// Fetch leaderboard with details
async function getLeaderboardWithDetails(count: number) {
  const topIds = await redis.zrevrange(
    "leaderboard:global", 0, count - 1, "WITHSCORES"
  );
  const pipeline = redis.pipeline();
  for (let i = 0; i < topIds.length; i += 2) {
    pipeline.hgetall(`player:${topIds[i]}`);
  }
  const details = await pipeline.exec();
  // Merge scores with details
}
```

### 4.5 Cache Invalidation Strategies

**Two-Tier Cache (Redis + Colyseus In-Memory):**

```typescript
// Colyseus room caches satellite data in memory
// Redis stores the canonical version with a version tag
async function getSatelliteData(fileName: string): Promise<any> {
  // Check in-memory cache first
  if (this.dataCache[fileName]) {
    return this.dataCache[fileName];
  }

  // Check Redis
  const data = await redis.get(`satellite:${fileName}`);
  if (data) {
    this.dataCache[fileName] = JSON.parse(data);
    return this.dataCache[fileName];
  }

  // Load from file, cache in Redis
  const fileData = await fs.readFile(`data/${fileName}`, "utf8");
  const parsed = JSON.parse(fileData);
  await redis.set(`satellite:${fileName}`, fileData, "EX", 3600);
  this.dataCache[fileName] = parsed;
  return parsed;
}

// When admin updates satellite data:
async function invalidateSatelliteData(fileName: string) {
  await redis.del(`satellite:${fileName}`);
  await redis.publish("imperio:cache:invalidate", fileName);
}

// All Colyseus processes listen for invalidation
subscriber.on("message", (channel, fileName) => {
  if (channel === "imperio:cache:invalidate") {
    delete this.dataCache[fileName];
  }
});
```

**TTL Strategy by Data Type:**

| Data | TTL | Reason |
|------|-----|--------|
| Session tokens | 1 hour | Security |
| Rate limit keys | 2 minutes | Auto-cleanup |
| Leaderboard cache | 5-15 minutes | Tolerable staleness |
| Satellite data | 1 hour (+ pub/sub invalidation) | Rarely changes |
| Reconnection tokens | 60 seconds | Per spec |
| Match state backups | Duration of match + 5 min | Crash recovery |

---

## 5. Turn-Based Game Design Patterns -- Framework-Agnostic

### 5.1 State Machine Design for Complex Phase Systems

**Explicit State Machine with Transition Table:**

```typescript
// Phase transition map -- the 11-phase state machine
const PHASE_TRANSITIONS: Record<Phase, Phase> = {
  LOBBY: "MATCH_INIT",
  MATCH_INIT: "REVEAL",
  REVEAL: "BID",
  BID: "INCOME_CALC",
  INCOME_CALC: "VISUAL_LEDGER",
  VISUAL_LEDGER: "ACTION",
  ACTION: "MOOD",        // or VOTE on election rounds
  MOOD: "DEALS",
  VOTE: "DEALS",          // VOTE replaces MOOD on election rounds
  DEALS: "EVENTS",
  EVENTS: "PRESS",
  PRESS: "AWARDS",        // or SNAPSHOT if not final round
  AWARDS: "SNAPSHOT",
  SNAPSHOT: "REVEAL",     // next round, or END if final
  END: "END",             // terminal
};

// Phase durations (ms)
const PHASE_DURATIONS: Record<Phase, number> = {
  REVEAL: 5000,
  BID: 30000,
  INCOME_CALC: 3000,
  VISUAL_LEDGER: 4000,    // fixed, server waits
  ACTION: 45000,
  MOOD: 20000,
  VOTE: 30000,
  DEALS: 30000,
  EVENTS: 5000,
  PRESS: 4000,
  AWARDS: 10000,
  SNAPSHOT: 2000,
};

function getNextPhase(current: Phase, context: GameContext): Phase {
  if (current === "ACTION") {
    return context.isElectionRound ? "VOTE" : "MOOD";
  }
  if (current === "PRESS") {
    return context.isFinalRound ? "AWARDS" : "SNAPSHOT";
  }
  if (current === "SNAPSHOT") {
    return context.roundNumber >= context.totalRounds ? "END" : "REVEAL";
  }
  return PHASE_TRANSITIONS[current];
}
```

**Phase Entry/Exit Hooks:**

```typescript
interface PhaseHandler {
  onEnter(state: ImperioState): void;
  onPlayerAction(state: ImperioState, playerId: string, data: any): void;
  onTimeout(state: ImperioState): void;
  onExit(state: ImperioState): void;
}

const phaseHandlers: Record<Phase, PhaseHandler> = {
  BID: {
    onEnter(state) {
      // Reset bid tracking
      state.owners.forEach(o => o.currentBid = 0);
    },
    onPlayerAction(state, playerId, data) {
      // Validate and record bid
      const owner = state.owners.get(playerId);
      if (data.amount > owner.cash) return; // reject
      owner.currentBid = data.amount;
    },
    onTimeout(state) {
      // No bid = no bid (per spec)
      this.resolveBids(state);
    },
    onExit(state) {
      // Cleanup
    },
  },
  // ... handlers for each phase
};
```

### 5.2 Deterministic Game Logic (Server-Authoritative)

**JIT Round Seed (per spec):**

```typescript
import { createHash } from "crypto";

function computeRoundSeed(
  matchSeed: string, serverSecret: string, roundNumber: number
): string {
  return createHash("sha256")
    .update(`${matchSeed}${serverSecret}${roundNumber}`)
    .digest("hex");
}

// Seeded PRNG for deterministic event selection
class SeededRNG {
  private seed: number;

  constructor(hexSeed: string) {
    this.seed = parseInt(hexSeed.substring(0, 8), 16);
  }

  // Mulberry32 -- fast, deterministic, sufficient for game logic
  next(): number {
    let t = this.seed += 0x6D2B79F5;
    t = Math.imul(t ^ t >>> 15, t | 1);
    t ^= t + Math.imul(t ^ t >>> 7, t | 61);
    return ((t ^ t >>> 14) >>> 0) / 4294967296;
  }

  // Pick index from weighted array
  weightedPick(weights: number[]): number {
    const total = weights.reduce((a, b) => a + b, 0);
    let random = this.next() * total;
    for (let i = 0; i < weights.length; i++) {
      random -= weights[i];
      if (random <= 0) return i;
    }
    return weights.length - 1;
  }
}
```

**All randomness MUST flow through the seeded RNG.** This guarantees:
- Same seed + same inputs = same outputs (replay capability)
- Server is the sole source of randomness (serverSecret never sent to client)
- Auditable: given the seed, any match can be replayed and verified

### 5.3 Event Sourcing vs. Snapshot-Based State

**Comparison for Imperio:**

| Aspect | Event Sourcing | Snapshot-Based |
|--------|---------------|---------------|
| **Storage** | All actions as events | Full state per round |
| **Replay** | Natural -- replay events | Reconstruct from snapshots |
| **Debugging** | See exact sequence of actions | See state at any point |
| **Storage Size** | Smaller per event, many events | Larger per snapshot, fewer snapshots |
| **Complexity** | Higher -- need event schema, projections | Lower -- just serialize state |
| **Undo** | Rewind by replaying up to N-1 | Restore previous snapshot |

**Recommendation for Imperio:** Use **hybrid approach**.
- **Snapshot-based** for the primary state sync (Colyseus already does this via `@colyseus/schema` delta encoding).
- **Event log** for audit trail and replay: store all player actions (bids, actions, votes, deals) as append-only events in PostgreSQL.
- **Round snapshots** at the SNAPSHOT phase for crash recovery and match history.

```typescript
// Event log entry (stored in PostgreSQL, not in Colyseus state)
interface GameEvent {
  eventId: string;         // UUID
  matchId: string;
  roundNumber: number;
  phase: Phase;
  playerId: string;
  eventType: string;       // "bid", "action", "deal_propose", "vote", etc.
  payload: Record<string, any>;
  timestamp: number;
}
```

### 5.4 Undo/Replay Systems

For a turn-based game with sequential phases, undo is typically NOT allowed once a phase advances (the server is authoritative). However, within a phase:

**In-Phase Undo (Before Submission):**
- Client-only concern. Let players change their bid/action selection before the timer expires.
- Only the final submission is sent to the server.
- Server ignores client-side undo -- it only processes the last message per phase per player.

**Match Replay:**
- Load round snapshots from PostgreSQL.
- Step through snapshots on the client to replay the match.
- Event log provides granular action-by-action replay if needed.

### 5.5 Spectator Mode Implementation

**Two-Room Pattern (option for clear separation):**

```typescript
// ImperioRoom -- for Owners (players)
class ImperioRoom extends Room<ImperioState> {
  maxClients = 12;  // Owners only

  onJoin(client, options) {
    if (this.state.owners.size >= 12) {
      throw new Error("Room full");
    }
    // Add as Owner
  }
}

// ImperioSpectatorRoom -- for Watchers
class ImperioSpectatorRoom extends Room<SpectatorState> {
  maxClients = 8;   // Up to 8 watchers (20 total - 12 owners)
  linkedRoomId: string;

  onCreate(options) {
    this.linkedRoomId = options.gameRoomId;
    // Subscribe to game room state updates via Redis pub/sub
    this.subscribeToGameRoom(this.linkedRoomId);
  }
}
```

**Single Room with Role Separation (simpler, recommended):**

```typescript
// Single room, but Watchers have limited interaction
onJoin(client, options) {
  const role = this.state.owners.size < 12 ? "owner" : "watcher";
  if (role === "watcher" && this.clients.length >= 20) {
    throw new Error("Room full");
  }
  // Watchers can view state but message handlers reject their actions
}

onMessage("bid", (client, data) {
  const owner = this.state.owners.get(client.sessionId);
  if (!owner) return; // Watchers can't bid -- silently ignore
  // ... process bid
});
```

### 5.6 Bot/AI for Disconnected Players

**Bot Activation on Disconnect:**

```typescript
// Activated after reconnection window expires (60s per spec)
class BotController {
  private rng: SeededRNG;

  constructor(roundSeed: string, ownerId: string) {
    this.rng = new SeededRNG(roundSeed + ownerId);
  }

  decideBid(property: PropertySchema, owner: OwnerSchema): number {
    // A8 heuristics: bid only if cash > GD 200K threshold
    if (owner.cash < 200_000) return 0;
    // Bid up to 80% of base price if property is in owned district
    const maxBid = Math.floor(property.basePrice * 0.8);
    return Math.min(maxBid, owner.cash);
  }

  decideAction(owner: OwnerSchema, properties: PropertySchema[]): Action {
    const ownedProps = properties.filter(p => p.ownerId === owner.ownerId);
    if (ownedProps.length === 0) return { type: "PASS" };

    // Prioritize upgrading cheapest property
    const cheapest = ownedProps.sort(
      (a, b) => a.upgradeLevel - b.upgradeLevel
    )[0];
    if (cheapest.upgradeLevel < 3 && owner.cash > 50_000) {
      return { type: "DEVELOP", propertyId: cheapest.propertyId };
    }

    return { type: "PASS" };
  }

  decideVote(): string {
    // Random vote among candidates
    const candidates = ["candidate_a", "candidate_b", "candidate_c"];
    return candidates[Math.floor(this.rng.next() * candidates.length)];
  }
}
```

**Key Rules for Bots:**
- Bots use the same code path as human players (send messages to the room).
- Bot decisions happen on the server -- no client needed.
- Bot behavior should be simple and predictable (not frustrating for remaining players).
- Bot actions are logged the same as human actions for replay/audit.

---

## 6. Mobile-First Game Architecture -- Performance Considerations

### 6.1 Network Optimization for Cellular Connections

**Message Batching:**
- Batch state updates at 200ms intervals instead of sending immediately. This halves battery drain.
- Colyseus `patchRate` of 50ms is fine for real-time; for a turn-based game, 200ms is acceptable.

```typescript
// Set a conservative patch rate for turn-based
this.setPatchRate(200); // 5 updates/sec instead of 20
```

**Payload Size:**
- Keep all messages under 4 KB. For Imperio, the typical message (bid, action, vote) is well under 1 KB.
- Use short field names in message payloads: `{ t: "bid", a: 50000, p: "prop_01" }` instead of `{ type: "bid", amount: 50000, propertyId: "prop_01" }`.

**Compression:**
- Colyseus schema uses binary encoding (delta compressed). Already efficient.
- For JSONB messages, consider MessagePack instead of JSON (40% smaller).

### 6.2 Battery-Efficient Polling/Push Strategies

**WebSocket Lifecycle Management:**

```gdscript
# Godot 4 -- align WebSocket with app lifecycle
func _notification(what):
    match what:
        NOTIFICATION_APPLICATION_FOCUS_OUT:
            # App backgrounded -- pause non-critical updates
            ws_manager.reduce_polling()
        NOTIFICATION_APPLICATION_FOCUS_IN:
            # App foregrounded -- resume
            ws_manager.resume_polling()
        NOTIFICATION_APPLICATION_PAUSED:
            # App fully paused -- close socket
            ws_manager.close()
        NOTIFICATION_APPLICATION_RESUMED:
            # App resumed -- reconnect
            ws_manager.reconnect()
```

**Heartbeat Configuration:**
- Ping/pong every 30 seconds (not every 5-10s -- saves battery).
- Colyseus has built-in heartbeat. Configure via transport options.

**Push over Poll:**
- Colyseus is already push-based (server sends state changes via WebSocket).
- Never poll the server from the client. Let the server push phase changes.

### 6.3 Offline-First Patterns for Reconnection

**Local State Queue:**

```gdscript
# Queue actions locally when offline
var pending_actions: Array = []

func submit_action(action: Dictionary):
    action["idempotencyKey"] = generate_uuid()
    if ws_manager.is_connected():
        ws_manager.send(action)
    else:
        pending_actions.append(action)
        save_pending_to_local_storage()

func _on_reconnected():
    # Replay pending actions
    for action in pending_actions:
        ws_manager.send(action)
    # Server handles idempotency -- duplicates are ignored
    pending_actions.clear()
    clear_local_storage()
```

**Sequence Numbers for Deduplication:**

```typescript
// Server-side: track last processed sequence per client
const lastSequence = new Map<string, number>();

onMessage("action", (client, data) {
  const lastSeq = lastSequence.get(client.sessionId) || 0;
  if (data.seq <= lastSeq) {
    // Duplicate -- already processed
    return;
  }
  lastSequence.set(client.sessionId, data.seq);
  // Process action
});
```

### 6.4 Progressive Loading for Game Assets

**Asset Loading Strategy:**

```
Phase 1 (immediate, <500KB):
  - Lobby UI
  - Basic typography and colors
  - Connection indicator

Phase 2 (background, <2MB):
  - Property card sprites
  - Player avatars
  - Board/map tiles

Phase 3 (lazy, on-demand):
  - Event card illustrations
  - Audio tracks per phase
  - Animation spritesheets
  - Dynasty emblems
```

**Godot 4 Background Loading:**

```gdscript
# ResourceLoader for async asset loading
func load_game_assets():
    var assets_to_load = [
        "res://assets/properties/property_cards.tres",
        "res://assets/map/lima_board.tscn",
        "res://assets/audio/ambient_lima.ogg",
    ]

    for path in assets_to_load:
        ResourceLoader.load_threaded_request(path)

func _process(_delta):
    # Check loading progress
    for path in loading_paths:
        var status = ResourceLoader.load_threaded_get_status(path)
        if status == ResourceLoader.THREAD_LOAD_LOADED:
            var resource = ResourceLoader.load_threaded_get(path)
            loaded_assets[path] = resource
            loading_paths.erase(path)
```

### 6.5 Memory Management on Mobile Devices

**Resource Budgets:**
- Target: 150MB total memory usage on mobile web
- WASM binary: 5-10MB (with stripped modules)
- Game assets: 20-30MB (loaded incrementally)
- State data: <1MB (Colyseus schema is compact)
- Audio: 10-20MB (streaming, not all loaded at once)

**Memory-Saving Techniques:**
1. **Texture atlases:** Pack property card sprites into atlases. Use `TextureRegion` for individual cards.
2. **Audio streaming:** Use OGG Vorbis for music (streamed), WAV for short SFX only.
3. **Scene pooling:** Reuse scene instances instead of instantiate/free cycles.
4. **Unload unused phases:** When transitioning from BID to INCOME_CALC, free BID scene resources.

```gdscript
# Unload previous phase resources
func _on_phase_exited(phase_name: String):
    if phase_name in loaded_phase_resources:
        for resource in loaded_phase_resources[phase_name]:
            resource.unreference()
        loaded_phase_resources.erase(phase_name)
```

### 6.6 Reconnection with Exponential Backoff

```gdscript
# Reconnection with exponential backoff
var reconnect_attempts: int = 0
var max_reconnect_attempts: int = 10
var base_delay_ms: float = 1000.0
var max_delay_ms: float = 30000.0

func _on_disconnected():
    attempt_reconnect()

func attempt_reconnect():
    if reconnect_attempts >= max_reconnect_attempts:
        show_connection_lost_dialog()
        return

    var delay = min(
        base_delay_ms * pow(2, reconnect_attempts),
        max_delay_ms
    )
    # Add jitter to prevent thundering herd
    delay += randf() * 1000.0

    reconnect_attempts += 1
    await get_tree().create_timer(delay / 1000.0).timeout

    var success = await ws_manager.reconnect()
    if success:
        reconnect_attempts = 0
    else:
        attempt_reconnect()
```

---

## Summary: Architecture Decision Matrix

| Decision | Recommendation | Rationale |
|----------|---------------|-----------|
| **State sync** | Colyseus @colyseus/schema | Built-in delta encoding, binary serialization, 64-field nesting |
| **Phase machine** | Explicit transition table + handler map | Clear, testable, matches spec's 11-phase structure |
| **Randomness** | Seeded PRNG (Mulberry32) with JIT round seed | Deterministic, replayable, auditable |
| **Client framework** | Godot 4.3+ with GDScript, raw WebSocket | Best web export support, no C# limitation |
| **Matchmaking** | filterBy() for casual, RankedQueueRoom for ranked | Colyseus native, GD wallet as rank proxy |
| **Persistence** | PostgreSQL hybrid (normalized + JSONB snapshots) | Queryable stats + full replay capability |
| **Currency** | Double-entry ledger in PostgreSQL per graf_dollar.txt | Idempotent, auditable, concurrent-safe |
| **Leaderboard** | Redis sorted sets (real-time) + PG materialized views (authoritative) | Fast reads + reliable aggregation |
| **Rate limiting** | Redis sliding window (sorted sets) | Per-spec limits, distributed across processes |
| **Reconnection** | 60s window + bot takeover + exponential backoff | Balances player experience with game flow |
| **Spectators** | Single room with role separation | Simpler than two-room; 20 maxClients covers both |
| **Asset loading** | Progressive: lobby first, game assets background | Mobile-friendly, fast time-to-interactive |
| **Data retention** | Monthly partitions + pg_partman + tablespace tiering | Automated, keeps hot data fast |
| **Scaling** | Redis presence + driver, PM2 processes, nginx LB | Colyseus native horizontal scaling pattern |

---

## Sources

### Colyseus
- [Colyseus Official Documentation](https://docs.colyseus.io/)
- [Colyseus GitHub Repository](https://github.com/colyseus/colyseus)
- [Colyseus State Best Practices](https://docs.colyseus.io/state/best-practices)
- [Colyseus Schema Definition](https://docs.colyseus.io/state/schema)
- [Colyseus Timing Events](https://docs.colyseus.io/server/room/timing-events)
- [Colyseus Scalability](https://docs.colyseus.io/deployment/scalability)
- [Colyseus Match-Maker API](https://docs.colyseus.io/server/matchmaker)
- [Colyseus Ranked Matchmaking](https://github.com/endel/colyseus-ranked-matchmaking)
- [Colyseus Command Pattern](https://docs.colyseus.io/recommendations/command-pattern)
- [Colyseus Lobby Room](https://docs.colyseus.io/room/built-in/lobby)
- [Multiplayer Game with Colyseus and PixiJS](https://arnauld-alex.com/guiding-the-flock-building-a-realtime-multiplayer-game-architecture-in-typescript)
- [How I Created an Online Multiplayer Game Using Colyseus](https://dev.to/konmaz/how-i-created-an-online-multiplayer-game-using-colyseus-npf)

### Godot
- [Godot WebSocket Documentation](https://docs.godotengine.org/en/stable/tutorials/networking/websocket.html)
- [Godot Web Export Documentation](https://docs.godotengine.org/en/stable/tutorials/export/exporting_for_web.html)
- [Godot Scene Organization](https://docs.godotengine.org/en/stable/tutorials/best_practices/scene_organization.html)
- [Godot Touch Input Manager (GDTIM)](https://github.com/Federico-Ciuffardi/GodotTouchInputManager)
- [godot-colyseus SDK](https://github.com/gsioteam/godot-colyseus)
- [Optimizing Godot Web Export Size](https://amann.dev/blog/2025/godot_web_size/)
- [Godot 4.3 Single-Threaded Web Export](https://forum.godotengine.org/t/godot-4-3-will-finally-fix-web-builds-no-sharedarraybuffers-required/38885)

### PostgreSQL
- [PostgreSQL Materialized Views](https://www.postgresql.org/docs/current/rules-materializedviews.html)
- [PostgreSQL for Gaming: Player Data and Leaderboards](https://reintech.io/blog/postgresql-for-gaming-player-data-leaderboards)
- [Database Architecture: History Over State](https://dev.to/mistval/database-architecture-history-over-state-3m8o)
- [Postgres JSON Optimization Techniques](https://markaicode.com/postgres-json-optimization-techniques-2025/)
- [pgledger: Ledger Implementation in PostgreSQL](https://www.pgrs.net/2025/03/24/pgledger-ledger-implementation-in-postgresql/)
- [Double-Entry Ledgers: The Missing Primitive](https://www.pgrs.net/2025/06/17/double-entry-ledgers-missing-primitive-in-modern-software/)
- [Designing Ledgers with Optimistic Locking](https://www.moderntreasury.com/journal/designing-ledgers-with-optimistic-locking)
- [Data Archiving and Retention in PostgreSQL](https://dataegret.com/2025/05/data-archiving-and-retention-in-postgresql-best-practices-for-large-datasets/)
- [Auto-Archiving with pg_partman](https://www.crunchydata.com/blog/auto-archiving-and-data-retention-management-in-postgres-with-pg_partman)
- [Time-Based Retention Strategies in Postgres](https://blog.sequinstream.com/time-based-retention-strategies-in-postgres/)

### Redis
- [Redis Sorted Sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/)
- [Redis Leaderboards](https://redis.io/solutions/leaderboards/)
- [Redis Rate Limiting](https://www.peakscale.com/redis-rate-limiting/)
- [Redis Pub/Sub](https://redis.io/glossary/pub-sub/)
- [Redis Cache Invalidation](https://redis.io/glossary/cache-invalidation/)
- [Redis Caching Strategies](https://dev.to/sepehr/redis-caching-strategies-from-basics-to-advanced-patterns-395n)
- [Managing Cache Invalidation with Redis Pub/Sub](https://osmos-tech-blog.medium.com/managing-in-memory-cache-invalidation-using-redis-pub-sub-c2bd60c13b69)
- [Building a Gaming Leaderboard with ElastiCache](https://aws.amazon.com/blogs/database/building-a-real-time-gaming-leaderboard-with-amazon-elasticache-for-redis/)

### Game Design Patterns
- [Game Programming Patterns: State](https://gameprogrammingpatterns.com/state.html)
- [Game Networking Demystified Series](https://ruoyusun.com/2019/03/28/game-networking-1.html)
- [Networking of a Turn-Based Game](https://longwelwind.net/blog/networking-turn-based-game/)
- [Making a Turn-Based Multiplayer Game in Rust](https://herluf-ba.github.io/making-a-turn-based-multiplayer-game-in-rust-01-whats-a-turn-based-game-anyway.html)
- [State Machines and Event Sourcing](https://gist.github.com/eulerfx/4ac420a14422ac960222)
- [State Management in Multiplayer Games](https://peerdh.com/blogs/programming-insights/state-management-patterns-in-multiplayer-game-development)

### Mobile Optimization
- [Building Games for Poor Mobile Connections](https://www.gamedeveloper.com/programming/building-games-that-run-on-poor-mobile-connections)
- [Optimizing for Mobile Networks (O'Reilly)](https://www.oreilly.com/library/view/high-performance-browser/9781449344757/ch08.html)
- [WebSocket Best Practices](https://ably.com/topic/websocket-architecture-best-practices)
- [WebSocket Testing for Mobile Apps](https://yrkan.com/blog/websocket-mobile-testing/)
- [WebSocket Performance in Android](https://moldstud.com/articles/p-achieving-high-performance-with-websockets-in-android-applications-a-comprehensive-guide)
