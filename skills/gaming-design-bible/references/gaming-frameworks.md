# Gaming Frameworks Reference

Technical patterns and best practices for the Colyseus + Godot 4.x + PostgreSQL + Redis stack, with focus on turn-based multiplayer game architecture.

---

## 1. COLYSEUS (Server Framework)

### Architecture Overview

Colyseus is a Node.js authoritative game server using WebSocket transport with schema-based state synchronization. Key concepts:

- **Room**: A game session instance. One room per match.
- **State**: Server-authoritative game state using `@colyseus/schema` for efficient binary serialization.
- **Client**: WebSocket connection with automatic reconnection support.
- **Matchmaker**: Routes players to rooms based on criteria.

### Room Lifecycle for Turn-Based Games

```
onCreate()     → Initialize state, set phase to LOBBY
onJoin()       → Add player to state, check if room is full
onMessage()    → Handle player actions (bid, action, deal, vote)
onLeave()      → Start reconnection timer (60s), then bot autopilot
onDispose()    → Clean up, save match snapshot to PostgreSQL
```

**Critical pattern for phase-based games:**

```typescript
// Phase state machine in room
class ImperioRoom extends Room<ImperioState> {
  private phaseTimer: Delayed;

  advancePhase() {
    const nextPhase = STATE_MACHINE[this.state.currentPhase];
    this.state.currentPhase = nextPhase;
    this.state.phaseDeadline = Date.now() + PHASE_DURATIONS[nextPhase];

    // Auto-advance when timer expires (server never waits for client)
    this.phaseTimer = this.clock.setTimeout(
      () => this.advancePhase(),
      PHASE_DURATIONS[nextPhase]
    );
  }
}
```

### State Schema Design

**Flat state trees perform better.** Deep nesting increases serialization overhead.

```typescript
// GOOD: Flat state with map collections
class ImperioState extends Schema {
  @type("string") currentPhase: string;
  @type("number") roundNumber: number;
  @type("number") phaseDeadline: number;
  @type({ map: OwnerSchema }) owners = new MapSchema<OwnerSchema>();
  @type({ map: PropertySchema }) properties = new MapSchema<PropertySchema>();
  @type([ "string" ]) marketRow = new ArraySchema<string>();
}

// BAD: Deeply nested state
class BadState extends Schema {
  @type(GameSchema) game; // game.round.phase.timer — 4 levels deep
}
```

**State size budget:** Keep total state under 50KB for 12-player rooms. Each schema change triggers delta serialization to all clients.

### Message Handling Patterns

```typescript
// Idempotent message handling
this.onMessage("bid", (client, data) => {
  // 1. Phase gating
  if (this.state.currentPhase !== "BID") return;

  // 2. Idempotency check
  if (this.processedKeys.has(data.idempotencyKey)) return;
  this.processedKeys.add(data.idempotencyKey);

  // 3. Validate
  const owner = this.state.owners.get(client.sessionId);
  if (!owner || owner.hasBid) return;
  if (data.amount > owner.cash) return;

  // 4. Apply (server-authoritative)
  owner.currentBid = data.amount;
  owner.hasBid = true;

  // 5. Check if all bids received → can advance early
  if (this.allOwnersBid()) this.resolvePhase();
});
```

### Reconnection Handling

```typescript
async onLeave(client: Client, consented: boolean) {
  if (consented) {
    // Player intentionally left — activate bot immediately
    this.activateBot(client.sessionId);
    return;
  }

  try {
    // Allow 60 seconds for reconnection
    const reconnection = this.allowReconnection(client, 60);
    // Bot takes over after 10s of disconnect
    const botTimer = this.clock.setTimeout(
      () => this.activateBot(client.sessionId),
      10_000
    );

    await reconnection;
    // Player reconnected — deactivate bot
    botTimer.clear();
    this.deactivateBot(client.sessionId);
  } catch {
    // Reconnection timed out — bot stays active
    this.activateBot(client.sessionId);
  }
}
```

### Matchmaking Strategy

```typescript
// ELO-band matchmaking for ranked modes
filterBy(options: any) {
  return {
    mode: options.mode,        // Quick/Standard/Long
    eloRange: Math.floor(options.elo / 200) * 200, // 200-point bands
  };
}
```

For casual matches, match by mode only. For ranked, use ELO bands with expanding search (wait 10s, then expand band by 100 points, repeat).

### Common Pitfalls

| Pitfall | Symptom | Solution |
|---------|---------|----------|
| State size bloat | Laggy clients, high bandwidth | Keep state flat, remove transient data after use |
| Synchronous DB calls in room | Room freezes during writes | Use async/await, write snapshots in onDispose |
| Missing idempotency | Duplicate actions on reconnect | Track processed `idempotencyKey` per room |
| No phase gating | Client sends actions for wrong phase | Validate `currentPhase` on every message |
| Timer drift | Phases end at wrong times | Use `this.clock` (Colyseus clock), not `setTimeout` |
| Memory leaks | Room memory grows over match | Clean up listeners in onDispose, clear timers |

### Scaling

- **Vertical:** Single Colyseus process handles ~100 concurrent rooms (depends on state size)
- **Horizontal:** Use `@colyseus/arena` or custom load balancer to distribute rooms across processes
- **Monitoring:** Built-in monitoring panel at `localhost:2567/colyseus`
- **Redis presence:** For multi-process deployments, use Redis for room discovery

---

## 2. GODOT 4.x (Client Framework)

### WebSocket Connection to Colyseus

Godot 4.x connects to Colyseus via WebSocket. For web export (WebAssembly), use the JavaScript bridge:

```gdscript
# WebSocketClient connection to Colyseus
var ws = WebSocketPeer.new()
var colyseus_url = "wss://server.example.com"

func _ready():
    ws.connect_to_url(colyseus_url)

func _process(_delta):
    ws.poll()
    while ws.get_available_packet_count() > 0:
        var packet = ws.get_packet()
        _handle_message(packet)
```

For production, use the Colyseus Godot SDK (if available) or a GodotJS bridge for the official JavaScript SDK.

### Scene Architecture for Phase-Based Games

```
Main
├── LobbyScene        (room browser, create/join, settings)
├── MatchScene         (core gameplay)
│   ├── MapView        (Lima district map, property markers)
│   ├── HUD            (cash, debt, fame, timer)
│   ├── PhasePanel     (changes per phase: bid panel, action panel, etc.)
│   ├── PropertyCards   (market row display, owned properties)
│   ├── VisualLedger   (animated cash flow display)
│   ├── DealPanel      (offer/counter deal interface)
│   ├── QuickChat      (12 preset buttons + speech bubbles)
│   └── EventOverlay   (dramatic event resolution animations)
├── ResultsScene       (end-of-match wealth reveal, share card)
└── ProfileScene       (GD balance, league badge, match history)
```

### Phase-Driven UI Pattern

```gdscript
# PhaseManager.gd — switches UI based on server phase
signal phase_changed(new_phase: String)

func _on_server_phase_change(phase: String):
    phase_changed.emit(phase)
    match phase:
        "BID":
            bid_panel.show()
            action_panel.hide()
        "ACTION":
            bid_panel.hide()
            action_panel.show()
        "VISUAL_LEDGER":
            visual_ledger.play_animation()
        # ... etc
```

### Mobile Input Best Practices

| Input Type | Godot Solution | Notes |
|-----------|---------------|-------|
| Text input | HTML DOM overlay | Godot virtual keyboard unreliable on mobile web |
| Bid amount | Preset buttons (+50K, +100K, +500K) | No typing required |
| Property selection | Tap on card/map marker | Large touch targets (48px minimum) |
| Quick Chat | 12 preset buttons in grid | No text input needed |
| Deal creation | Template-based UI with dropdowns | Server-generated suggestions reduce friction |
| Scrolling | Swipe gestures on card lists | Use ScrollContainer with touch support |

### Animation System for Board Games

Visual feedback is critical for turn-based games where "nothing is happening" between decisions:

1. **Visual Ledger** (INCOME_CALC phase):
   - Cash flows animated as particle streams between player icons
   - Duration: 4.0 seconds (server waits)
   - Color coding: green = income, red = expense, gold = rent

2. **Auction countdown**:
   - Timer circle fills as deadline approaches
   - Last 5 seconds: heartbeat pulse effect + sound
   - Winning bid: gold burst particle effect

3. **Event strike**:
   - Newspaper headline slides in from top
   - Target player's avatar pulses red
   - Damage numbers float up (red for loss, green for mitigation)

4. **Property acquisition**:
   - Card slides from Market Row to player's portfolio
   - District map marker lights up in player's color

5. **Deal resolution**:
   - Split-screen showing both parties
   - Assets slide between players
   - Handshake animation on completion

### WebAssembly Export Notes

| Limitation | Workaround |
|-----------|------------|
| No native file access | Use JavaScript bridge for saves (localStorage) |
| Thread limitations | Avoid multithreading, use async patterns |
| Audio autoplay blocked | Require user interaction before first audio |
| Large initial download | Progressive loading: core first, assets stream |
| Text input unreliable | HTML DOM overlay for all text fields |
| No clipboard access | JavaScript bridge for share card copy/save |
| Memory constraints | Target < 256MB total, unload unused scenes |

### Audio Design System

```
Layer 1: Ambient     (district-themed background loop, low volume)
Layer 2: Phase music  (subtle change per phase — tension builds in BID)
Layer 3: Event stings (orchestral hits for events, auctions, deals)
Layer 4: UI feedback  (button clicks, card movements, coin sounds)
```

- Cross-fade between ambient tracks when district focus changes
- sfx_cash_loss: Low orchestral sting + paper shuffle (NOT sad trombone)
- sfx_auction_win: Brass fanfare + crowd murmur
- sfx_event_strike: Timpani hit + tension strings

---

## 3. POSTGRESQL (Persistence Layer)

### Schema for Game Data

```sql
-- Match history with JSONB snapshots
CREATE TABLE matches (
    match_id    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    mode        VARCHAR(10) NOT NULL,  -- 'quick', 'standard', 'long'
    created_at  TIMESTAMPTZ DEFAULT now(),
    ended_at    TIMESTAMPTZ,
    final_state JSONB,  -- Complete end-of-match state snapshot
    round_count SMALLINT
);

-- Per-player match results (denormalized for fast leaderboard queries)
CREATE TABLE match_results (
    match_id    UUID REFERENCES matches(match_id),
    user_id     UUID REFERENCES users(user_id),
    final_rank  SMALLINT NOT NULL,
    final_wealth BIGINT NOT NULL,  -- GD amount
    gd_earned   BIGINT NOT NULL,   -- Total GD credited to wallet
    PRIMARY KEY (match_id, user_id)
);

-- GD Ledger (double-entry per graf_dollar.txt)
CREATE TABLE ledger_tx (
    tx_id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tx_type         VARCHAR(20) NOT NULL,  -- 'award', 'transfer', 'decay'
    idempotency_key VARCHAR(120) UNIQUE NOT NULL,
    created_at      TIMESTAMPTZ DEFAULT now(),
    description     TEXT
);

CREATE TABLE ledger_entry (
    entry_id    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tx_id       UUID REFERENCES ledger_tx(tx_id),
    account_id  UUID NOT NULL,  -- user_id or 'SYSTEM'
    amount      BIGINT NOT NULL,  -- positive = credit, negative = debit
    balance     BIGINT NOT NULL,  -- running balance after this entry
    CONSTRAINT entries_balance CHECK (balance >= 0 OR account_id = 'SYSTEM')
);
```

### Leaderboard Queries at Scale

```sql
-- Materialized view for leaderboard (refresh every 5 minutes)
CREATE MATERIALIZED VIEW leaderboard AS
SELECT
    u.user_id,
    u.display_name,
    u.dynasty_id,
    COALESCE(le.balance, 0) as gd_balance,
    CASE
        WHEN COALESCE(le.balance, 0) >= 200000000 THEN 'emperador'
        WHEN COALESCE(le.balance, 0) >= 50000000  THEN 'magnate'
        WHEN COALESCE(le.balance, 0) >= 10000000  THEN 'inversionista'
        WHEN COALESCE(le.balance, 0) >= 1000000   THEN 'propietario'
        ELSE 'inquilino'
    END as league,
    RANK() OVER (ORDER BY COALESCE(le.balance, 0) DESC) as global_rank
FROM users u
LEFT JOIN LATERAL (
    SELECT balance FROM ledger_entry
    WHERE account_id = u.user_id
    ORDER BY entry_id DESC LIMIT 1
) le ON true;

CREATE UNIQUE INDEX ON leaderboard (user_id);
CREATE INDEX ON leaderboard (gd_balance DESC);
CREATE INDEX ON leaderboard (dynasty_id, gd_balance DESC);

-- Refresh strategy
REFRESH MATERIALIZED VIEW CONCURRENTLY leaderboard;
```

### Match Snapshot Strategy

Store full match state as JSONB at end of match:

```sql
-- Full state snapshot (for replay, debugging, analytics)
UPDATE matches SET
    final_state = $1::jsonb,  -- Complete ImperioState serialized
    ended_at = now()
WHERE match_id = $2;
```

**What to store:**
- Final state of all owners (wealth, properties, upgrades, fame)
- Complete event history (which events hit whom)
- Deal history (all offers, acceptances, rejections)
- Round-by-round wealth progression (for charts)
- Winning bid amounts per round (for economy analytics)

**What NOT to store:**
- Phase timing data (not useful for analysis)
- Client-side animation states
- WebSocket connection metadata

### Seasonal Decay Implementation

```sql
-- Monthly decay: 10% above GD 5,000,000
-- Run as scheduled job (cron/pg_cron)
WITH decay_targets AS (
    SELECT DISTINCT account_id, balance
    FROM ledger_entry le
    WHERE account_id != 'SYSTEM'
    AND balance > 5000000
    AND NOT EXISTS (
        SELECT 1 FROM ledger_entry le2
        WHERE le2.account_id = le.account_id
        AND le2.entry_id > le.entry_id
    )
),
decay_amounts AS (
    SELECT
        account_id,
        balance,
        FLOOR((balance - 5000000) * 0.10) as decay_amount
    FROM decay_targets
)
-- Insert decay transactions via application layer (not raw SQL)
-- to maintain double-entry ledger integrity
SELECT * FROM decay_amounts WHERE decay_amount > 0;
```

### Indexing Strategy

| Table | Index | Purpose |
|-------|-------|---------|
| ledger_entry | `(account_id, entry_id DESC)` | Latest balance lookup |
| ledger_tx | `(idempotency_key) UNIQUE` | Duplicate prevention |
| match_results | `(user_id, final_wealth DESC)` | Personal best queries |
| match_results | `(final_rank, gd_earned DESC)` | Win rate analytics |
| users | `(dynasty_id)` | Dynasty member lookups |

---

## 4. REDIS (Real-Time Layer)

### Real-Time Leaderboard

```
# Sorted set for instant leaderboard queries
ZADD leaderboard:global {gd_balance} {user_id}
ZADD leaderboard:dynasty:{dynasty_id} {gd_balance} {user_id}

# Top 100
ZREVRANGE leaderboard:global 0 99 WITHSCORES

# Player's rank
ZREVRANK leaderboard:global {user_id}

# Players around you (context leaderboard)
ZREVRANK leaderboard:global {user_id}  → rank
ZREVRANGE leaderboard:global {rank-5} {rank+5} WITHSCORES
```

### Rate Limiting

```
# Sliding window rate limiter
# Key: ratelimit:{user_id}:{action}
# Example: max 1 bid per round, max 5 reconnects per minute

MULTI
ZADD ratelimit:{user_id}:bid {timestamp} {request_id}
ZREMRANGEBYSCORE ratelimit:{user_id}:bid 0 {timestamp - window_ms}
ZCARD ratelimit:{user_id}:bid
EXEC
# If ZCARD > limit, reject the request
```

### Session Management

```
# Store session for reconnection
SET session:{session_id} {json_data} EX 3600  # 1 hour TTL

# Room presence (which rooms are active)
SADD rooms:active {room_id}
SREM rooms:active {room_id}

# Player online status
SET online:{user_id} {room_id} EX 300  # 5 min TTL, refreshed by heartbeat
```

### Pub/Sub for Cross-Room Events

```
# Dynasty notifications
PUBLISH dynasty:{dynasty_id} '{"type":"member_win","user":"PlayerName","wealth":3500000}'

# Friend online notifications
PUBLISH friends:{user_id} '{"type":"online","friend_id":"..."}'

# Colyseus room subscribes to relevant channels
```

### Cache Strategy

| Data | Cache In Redis? | TTL | Invalidation |
|------|----------------|-----|-------------|
| Leaderboard top 100 | Yes | 5 min | On materialized view refresh |
| Player profile | Yes | 15 min | On match end |
| Dynasty info | Yes | 30 min | On member join/leave |
| Match state | No | — | Colyseus handles in-memory |
| Property deck | Yes | 24 hours | On deck update |
| Event deck | Yes | 24 hours | On event update |

---

## 5. TURN-BASED GAME DESIGN PATTERNS

### Deterministic State Machine

All game logic MUST be deterministic given the same inputs:

```typescript
// Round seed ensures reproducible event selection
const roundSeed = sha256(`${matchSeed}${serverSecret}${roundNumber}`);

// Deterministic random from seed
function seededRandom(seed: string, index: number): number {
    return (sha256(`${seed}:${index}`) % 10000) / 10000;
}
```

**Why deterministic?**
- Replay system: recreate any match from seed + player inputs
- Debugging: reproduce exact game state from logs
- Anti-cheat: server and client should reach same state
- Event sourcing: rebuild state from action log

### Event Sourcing vs Snapshots

| Approach | Pros | Cons | Use When |
|----------|------|------|----------|
| **Event sourcing** | Full replay, audit trail, debugging | Complex, storage grows linearly | Need replay/audit capability |
| **Snapshots** | Simple, fast state load, small storage | No replay, lossy | MVP, simple matches |
| **Hybrid** | Best of both | More complex implementation | Production systems |

**Recommended for IMPERIO:** Hybrid — snapshot at end of each round (for quick state restoration on reconnect), plus event log for full match replay capability.

### Spectator Mode Implementation

```
Spectators receive:
  - Full state snapshots (same as players)
  - Phase change notifications
  - Event resolution animations

Spectators do NOT receive:
  - Sealed bid amounts before reveal
  - Deal offers before resolution
  - Internal RiskScore calculations

Spectators CAN:
  - Vote on Moods
  - Send Quick Chat (displayed differently)
  - Spread Rumors (once per match)
```

**Technical pattern:** Spectators join the same Colyseus room but with `isSpectator: true` flag. Server filters state deltas to exclude private info until reveal phase.

### Bot System Architecture

```typescript
interface BotHeuristic {
    // Bid decision: how much to bid on a property
    decideBid(property: Property, owner: Owner, gameState: State): number;

    // Action decision: which major + minor action to take
    decideAction(owner: Owner, gameState: State): { major: Action, minor?: Action };

    // Deal decision: accept or reject incoming deal
    evaluateDeal(deal: Deal, owner: Owner, gameState: State): boolean;
}

// Difficulty tiers
const BOT_CONFIGS = {
    tutorial: { bidAggressiveness: 0.3, developThreshold: 0.5 },
    easy:     { bidAggressiveness: 0.5, developThreshold: 0.4 },
    medium:   { bidAggressiveness: 0.7, developThreshold: 0.3 },
    hard:     { bidAggressiveness: 0.9, developThreshold: 0.2 },
};
```

---

## 6. MOBILE-FIRST ARCHITECTURE

### Network Optimization

| Technique | Implementation | Savings |
|-----------|---------------|---------|
| Binary serialization | `@colyseus/schema` (default) | ~70% vs JSON |
| Delta compression | Colyseus sends only changed fields | ~90% reduction |
| Message batching | Batch phase change + state update | Fewer round-trips |
| Heartbeat interval | 15s (not default 1s) for mobile | Battery savings |
| Reconnection buffer | Queue messages during reconnect | No data loss |

### Battery-Efficient Design

- WebSocket with 15s heartbeat (not 1s)
- No polling loops — use server push exclusively
- Reduce animation frame rate when app is backgrounded
- Pause non-essential audio when screen off
- Coalesce state updates (don't process every delta individually)

### Offline-First Reconnection

```
1. Connection lost → show "Reconnecting..." overlay
2. Queue any pending player actions locally
3. Attempt reconnection every 2s, exponential backoff to 30s
4. On reconnect: send queued actions, receive full state sync
5. After 60s: bot takes over, player notified via push notification
6. Player can rejoin from push notification → resume from bot
```

### Progressive Loading

```
Phase 1 (< 2s):  Core UI shell, connection to server
Phase 2 (< 5s):  Game state sync, player data, current phase UI
Phase 3 (< 10s): Map textures, property card art, ambient audio
Phase 4 (lazy):  Animation assets, achievement badges, profile data
```

### Memory Management

- Unload Results/Profile scenes when in match
- Property card images: load visible cards only, recycle off-screen
- Event animations: load on demand, unload after play
- Audio: stream ambient, preload only current-phase stings
- Target: < 200MB total memory usage on mobile
