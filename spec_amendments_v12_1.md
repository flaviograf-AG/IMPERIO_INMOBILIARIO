# IMPERIO INMOBILIARIO — Spec Amendments v12.1

**Source:** Gaming Design Bible audit (2026-02-04)
**Applies to:** `imperio_master_spec_v11.md` (v12.0)
**Scope:** All CRITICAL and HIGH findings from the cross-reference audit

---

## HOW TO READ THIS DOCUMENT

Each amendment is marked with:
- **[REPLACE]** — Delete the existing section and insert this text
- **[INSERT AFTER]** — Add this text after the referenced line/section
- **[AMEND]** — Modify specific lines within the existing section

---

## CHANGELOG (v12.0 → v12.1)

### Economy Rebalance
- **OpsCost formula**: Increased from `/4 × 8K` to `/3 × 12K` for meaningful portfolio friction
- **Leader friction tax**: Changed from flat GD amounts to percentage-based rent tax
- **Late-game cash sinks**: Added multi-level Develop (levels 3-5) and final-round prestige auction
- **Deal price validation**: Added floor/ceiling rules to prevent kingmaking via asset dumping

### Retention & Virality
- **Challenge system**: Added daily/weekly/seasonal challenge tiers with GD rewards
- **Matchmaking**: Added Part 14 — matchmaking, room discovery, bot-fill rules
- **Dynasty overhaul**: Evolved from cosmetic labels to social groups with chat and collective goals
- **Portada upgrade**: Per-player share cards, challenge CTA, deep link architecture, multi-trigger
- **Streamer support**: Added streamer mode, clip export, anonymous spectating
- **Referral rework**: Multi-tier GD rewards replacing single-tier Dynasty XP
- **Returning players**: Added "Bono de regreso" re-engagement mechanic

### Technical
- **Reconnection flow**: Bot activation at 10s (not 60s), state resync protocol
- **Bot heuristics**: Added H6-H9 for Mood/Vote, Packaging, Fame spend, deal evaluation
- **Event AutoChoice**: Replaced with 10s player choice window + timeout fallback

---

# AMENDMENT C1: MATCHMAKING SPECIFICATION

**Action:** [INSERT AFTER] Part 13 (line 1491), before `# APPENDICES`

```markdown
---

# PART 14: MATCHMAKING & ROOM DISCOVERY

## 14.1 Room Discovery

### 14.1.1 Lobby Browser (PUBLIC Rooms)

Players see a filterable list of available PUBLIC rooms:

| Filter | Options |
|--------|---------|
| Mode | Quick / Standard / Long |
| Status | Waiting / In Progress (spectate only) |
| Players | Show current/max (e.g., "4/12") |
| Sort | Newest first (default), Most players, Starting soon |

**Refresh:** Auto-refresh every 5s. Manual pull-to-refresh on mobile.

**Create Room:** One-tap "Crear Partida" button with mode selection (Quick default).

### 14.1.2 Quick Play (Auto-Match)

"Jugar Ya" button — auto-joins the best available room:

```
1. Filter rooms by selected mode (Quick default)
2. Prefer rooms with 3+ players (close to starting)
3. If no suitable room exists, create a new one
4. Player enters LOBBY state immediately
```

### 14.1.3 Friends Room (Private)

```
1. Host taps "Crear Sala Privada" → selects mode
2. Server generates 5-character alphanumeric room code (e.g., LIMA7)
3. Host shares via:
   - One-tap "Compartir" → WhatsApp (primary LATAM), Telegram, Copy Link
   - Share message: "Juega Imperio Inmobiliario conmigo! Codigo: {CODE} — imperio.game/join/{CODE}"
   - QR code display (for in-person invites)
4. Friends join via:
   - Deep link: opens app/web client directly into room
   - Manual: "Unirse con Codigo" input field in lobby
```

## 14.2 Match Start Rules

| Rule | Value |
|------|-------|
| Minimum players to start | 4 Owners |
| Maximum Owners | 12 (Standard), 8 (Quick), 12 (Long) |
| Maximum participants | 20 (Owners + Watchers) |
| Host start | Host can start manually when ≥ 4 Owners |
| Auto-start | 30s countdown when room reaches max Owners |
| Bot fill (PUBLIC) | If < 4 Owners after 120s in LOBBY, fill remaining with bots up to 6 total |
| Bot fill (FRIENDS) | Host-controlled: toggle "Rellenar con bots" (default OFF) |
| Late join | Allowed during Round 1 only (fill empty seats) |

## 14.3 Matchmaking Tiers (Phase 2 — Ranked Mode)

Ranked mode unlocks at Propietario league (GD 1,000,000+).

```
filterBy:
  mode: selected_mode
  eloRange: floor(player_elo / 200) * 200

searchExpansion:
  0-10s:  exact eloRange match
  10-20s: ±1 eloRange band (±200 ELO)
  20-30s: ±2 eloRange bands (±400 ELO)
  30s+:   any rank, start with bots if needed
```

**ELO Calculation:**
- Starting ELO: 1000
- Win: +25 × (1 - expected_score)
- Loss: -25 × expected_score
- expected_score = 1 / (1 + 10^((opponent_avg_elo - player_elo) / 400))
- Use average opponent ELO for multiplayer calculation

## 14.4 Deep Link Architecture

| Purpose | URL Pattern | Behavior |
|---------|------------|----------|
| Room invite | `imperio.game/join/{roomCode}` | Opens client, joins room directly |
| Referral | `imperio.game/ref/{userId}` | Opens client, registers referrer, starts onboarding |
| Rematch | `imperio.game/play/{matchId}` | Opens client, creates room with same mode/players invited |
| Dynasty | `imperio.game/dynasty/{dynastyId}` | Opens client, shows dynasty page with join option |
| Spectate | `imperio.game/watch/{roomId}` | Opens client as Watcher in active match |

**Non-player flow:**
1. Link opens web client (no install required for WASM build)
2. If no account: spectate first match anonymously
3. After match ends: CTA "Crea tu cuenta y juega" with one-tap signup
4. Account creation pre-fills referrer if deep link included `ref/` parameter
```

---

# AMENDMENT C2: RETENTION LOOPS & CHALLENGE SYSTEM

**Action:** [REPLACE] Section 3.5 (lines 292-327)

```markdown
## 3.5 Viral & Retention Hooks

### 3.5.1 Share Assets

| Asset | Dimensions | Trigger |
|-------|------------|---------|
| Portada | 1200×630 + 1080×1920 | Match end (ALL players, not just winner) |
| Trophy (Logro) | 1080×1920 | Mid-match achievement |
| League Card | 1080×1920 | League promotion |
| Streak Card | 1080×1920 | 3+ win streak |
| Dynasty Card | 1200×630 | Dynasty milestone (top 10 weekly) |

See Appendix D for layouts and required elements.

### 3.5.2 Competitive Systems
- **Dynasty Ladder**: Weekly seasons with leaderboard
- **Revancha al toque**: Quick rematch flow (see Part 14 deep links)

### 3.5.3 Watcher Awards
- "Favorito del publico" (voted by Watchers)
- "Ampay del dia" (scandal highlight)

### 3.5.4 Referral System (REVISED in v12.1)

Multi-tier rewards to drive viral growth:

| Milestone | Referrer Reward | Referee Reward |
|-----------|----------------|----------------|
| Account creation | +GD 25,000 (persistent) | +GD 10,000 welcome bonus (persistent) |
| First match completed | +GD 50,000 (persistent) | Tutorial completion badge |
| Reach Propietario league | +GD 100,000 (persistent) | — |

**Milestones:**
- 5 referrals: Exclusive "Reclutador" badge
- 10 referrals: Exclusive "Padrino" badge
- 20 referrals: Exclusive "Magnate Social" badge

**Limits:** 20 referral rewards per calendar month per player (anti-abuse).

**Tracking:** Server records `referredBy` on account creation. Milestones tracked in `user_referrals` table.

### 3.5.5 Streak System (REVISED in v12.1)

- **Daily Login**: +GD 5,000 (persistent) base reward
- **Streak Multiplier**: Day 2 = 1.5x, Day 3 = 2x, Day 7+ = 3x
- **Streak Break**: Resets to Day 1 after 48h without login
- **Returning Player Bonus** ("Bono de regreso"): After 7+ days without playing a match, next 3 matches earn 2x persistent GD. Notification: "Tu dinastia te extrana. Vuelve y gana el doble."

### 3.5.6 Challenge System (NEW in v12.1)

Challenges defined in `data/challenges.json` as satellite data.

#### Daily Challenges (3 active, rotate at 00:00 UTC-5)

| Example Challenge | Reward |
|-------------------|--------|
| "Gana una subasta" (Win an auction) | +GD 15,000 (persistent) |
| "Cierra un trato" (Complete a deal) | +GD 15,000 (persistent) |
| "Acumula GD 500K en un match" (Earn 500K in one match) | +GD 25,000 (persistent) |
| "Crea un paquete inmobiliario" (Create a package) | +GD 20,000 (persistent) |
| "Vota en la eleccion" (Vote in an election) | +GD 10,000 (persistent) |
| "Juega una Partida Rapida" (Play a Quick Match) | +GD 10,000 (persistent) |

**Pool:** 15+ rotating daily challenges. Server selects 3 per day, no repeats within 3 days.

**Completion:** Tracked server-side per match events. Challenge progress persists across matches within the day.

#### Weekly Dynasty Challenge (1 active, resets Monday 00:00 UTC-5)

Collective goal shared by all dynasty members:

| Example Challenge | Reward (per member) |
|-------------------|---------------------|
| "La dinastia gana 30 partidas" (Dynasty wins 30 matches) | +GD 100,000 + exclusive weekly badge |
| "Miembros acumulan GD 50M total" (Members earn 50M total) | +GD 150,000 |
| "5 miembros alcanzan top 3 en un match" (5 members reach top 3) | +GD 100,000 + dynasty crest upgrade |

**Pool:** 8+ rotating weekly challenges.

**Progress:** Dynasty-wide progress bar visible in lobby. Dynasty chat notifications at 50% and 90%.

#### Seasonal Goals (1 per season, ~30 days)

Tied to league progression:

| Goal | Reward |
|------|--------|
| Reach next league tier this season | Exclusive seasonal badge (unique per season) |
| Play 50 matches this season | Seasonal title ("Veterano de [Season Name]") |
| Win 10 matches this season | Exclusive portrait frame |

### 3.5.7 Leaderboard System

Lifetime GD leaderboard with tiered leagues. See **Part 13** for full specification.

- **5 league tiers**: Inquilino → Propietario → Inversionista → Magnate → Emperador
- **Monthly seasonal decay**: balances > GD 5M decay 10% monthly
- **Dynasty leaderboard**: aggregate team GD (strongest viral hook — recruit friends)
- **Contextual views**: Friends, Regional (Lima districts), Dynasty, All-time
```

---

# AMENDMENT H1: OPSCOST FORMULA

**Action:** [REPLACE] Section 6.5 (lines 777-785)

```markdown
## 6.5 Operating Costs (Portfolio Overhead)

```
OpsCost = floor((NumProperties + TotalUpgrades) / 3) × 12,000
```

Properties in a Package count as `ceil(count/2)` toward NumProperties for OpsCost (efficiency bonus).

**Impact table:**

| Properties | Upgrades | Raw Count | OpsCost (GD) | Typical Rent | OpsCost % of Rent |
|------------|----------|-----------|-------------|--------------|-------------------|
| 1 | 0 | 1 | 0 | 30-50K | 0% |
| 2 | 0 | 2 | 0 | 60-100K | 0% |
| 3 | 0 | 3 | 12,000 | 100-150K | 8-12% |
| 4 | 2 | 6 | 24,000 | 150-220K | 11-16% |
| 6 | 4 | 10 | 36,000 | 250-350K | 10-14% |
| 8 | 6 | 14 | 48,000 | 350-500K | 10-14% |
| 3 (pkg) | 0 | 2 | 0 | 120K+ | 0% (package benefit) |
| 5 (3 pkg+2) | 2 | 6 | 24,000 | 200-280K | 9-12% |

**Design rationale:** OpsCost should consume 10-15% of rent income at 4+ properties to create genuine friction for portfolio expansion. The previous formula (`/4 × 8K`) produced 3-5% friction — cosmetic, not strategic.

Applied during INCOME phase after rent collection, before leader friction tax.
```

---

# AMENDMENT H2: LEADER FRICTION TAX

**Action:** [REPLACE] Section 6.6 (lines 787-792)

```markdown
## 6.6 Leader Friction ("Impuesto a la fama")

Applied during INCOME phase after OpsCost:

```
LeaderTax:
  Top Wealth quartile: 5% of TotalRentThisRound
  #1 Wealth (if Wealth > 1.2 × Wealth(#2)): 10% of TotalRentThisRound
```

| Scenario | Rent Income | Tax | Effective Rent |
|----------|-------------|-----|----------------|
| Top quartile, GD 150K rent | 150,000 | 7,500 | 142,500 |
| Top quartile, GD 300K rent | 300,000 | 15,000 | 285,000 |
| #1 runaway, GD 300K rent | 300,000 | 30,000 | 270,000 |
| #1 runaway, GD 500K rent | 500,000 | 50,000 | 450,000 |

**Design rationale:** Percentage-based tax scales with success. A flat GD 8-15K tax was 5-10% of early-game income but <3% of late-game income — invisible to leaders. The 5%/10% split creates genuine tension: getting rich is still rewarded, but the tax is felt in every INCOME calculation. Reference: Power Grid's turn-order reversal achieves similar friction at ~8-12% effective disadvantage.

**INCOME phase order:** Rent → Signals → OpsCost → Leader Tax → Debt repayment → Net cash change.
```

---

# AMENDMENT H3: LATE-GAME CASH SINKS

**Action:** [INSERT AFTER] Section 6.7 (line 798), before `---` and Part 7

```markdown
## 6.8 Late-Game Cash Sinks (Prevent Inflation)

### 6.8.1 Multi-Level Development

Properties can be developed beyond level 2 (requires prior upgrade to each level):

| Level | Cost (GD) | Rent Bonus | Conditions |
|-------|----------|------------|------------|
| 1 | 60,000-100,000 | +12,000/round | Standard (existing) |
| 2 | 60,000-100,000 | +12,000/round | Standard (existing) |
| 3 | 150,000 | +15,000/round | Available from Round 5+ |
| 4 | 200,000 | +18,000/round | Available from Round 8+ |
| 5 | 300,000 | +20,000/round | Long Mode only (Round 10+) |

**Design rationale:** Levels 3-5 provide diminishing returns per GD invested (+10% rent/GD at level 3 vs +15-20% at levels 1-2) but give wealthy players a productive use for accumulated cash. Without these sinks, players accumulate GD 1-2M liquid cash with no meaningful use in rounds 8+.

### 6.8.2 Prestige Auction (Final Round Only)

In the final round's ACTION phase, an additional Major action is available:

**"Inversion de Prestigio"** — Bid any amount of cash. The bid is removed from your in-match Wealth but converts to a **1.5x multiplier** on the bid amount for persistent GD earnings.

```
Example: Player bids GD 500,000
  In-match: -GD 500,000 Wealth (affects final ranking)
  Persistent: +GD 750,000 additional persistent GD (on top of normal match earnings)
```

**Trade-off:** Spending cash hurts your in-match final Wealth ranking (and placement bonus) but earns more persistent GD. This creates a genuine dilemma: finish 1st in the match, or sacrifice rank for long-term leaderboard climbing?

**Limit:** Maximum bid = 50% of current cash (prevents all-in dumps that distort rankings).
```

---

# AMENDMENT H4: EVENT CHOICE WINDOW

**Action:** [REPLACE] Section 8.4 (lines 1072-1089)

```markdown
## 8.4 Event Resolution (Player Choice with Timeout)

When an event offers a choice (pay cash to avoid penalty vs. accept penalty):

### 8.4.1 Choice Window

```
1. Server presents event with options to targeted Owner
2. Owner has 10 seconds to choose
3. If Owner chooses within 10s: apply chosen option
4. If timeout (no response): apply AutoChoice fallback
```

### 8.4.2 AutoChoice Fallback (Timeout Only)

```python
if choice.payCash <= player.cash:
    if choice.payCash <= choice.penaltyValue:
        take "pay" option
    else:
        take "penalty" option
else:
    take "penalty" option
```

### 8.4.3 Pre-Match Preference (Optional Override)

Players can set a default preference before the match:
- "Siempre pagar para evitar penalidades" (Always pay to avoid penalties)
- "Siempre aceptar la penalidad" (Always accept penalty)
- "Decidir en el momento" (Decide in real-time) — **default**

If preference is set to "always pay" or "always accept," the server auto-resolves without the 10s window. Player can change preference between rounds via settings.

### 8.4.4 Design Rationale

Events are one of the game's most dramatic moments — a share-worthy "will they pay?" beat. Auto-resolving by default kills both tension and shareability. The 10s window is short enough to maintain game pace while giving players agency over their most impactful financial decisions. Players who prefer speed can opt into auto-resolve via pre-match preference.

**Phase timing:** The EVENTS phase timer accommodates the 10s choice window. If multiple events resolve in one round (rare, global + targeted), choices are sequential with 10s each.
```

---

# AMENDMENT H5: STREAMER & CONTENT CREATOR SUPPORT

**Action:** [INSERT AFTER] Section 3.5 (after the new 3.5.7), before Part 4

```markdown
### 3.5.8 Streamer Mode (NEW in v12.1)

Toggle in player settings. When active:

| Element | Normal Mode | Streamer Mode |
|---------|-------------|---------------|
| Cash display | Exact GD amount | Tier label ("Rico", "Medio", "Apretado") |
| Bid amounts | Exact | Hidden until resolution |
| Player names | Custom names | Custom names (unchanged) |
| UI layout | Standard | Stream-overlay-friendly (wider margins, higher contrast) |
| Delay option | None | 5s / 10s configurable stream delay |

**Stream delay:** When enabled, all state updates are buffered client-side for the configured delay before rendering. Prevents stream-sniping in competitive matches.

### 3.5.9 Clip Export (NEW in v12.1)

One-tap clip generation from key match moments:

**Clip-worthy triggers** (server marks these in match timeline):
- Auction win (especially last-second steals)
- Event strike (dramatic event resolution)
- Deal completion (especially betrayals or large trades)
- Wealth overtake (#1 position change)
- League promotion (end-of-match)
- Package creation

**Clip spec:**
- Duration: 10-15 seconds centered on trigger moment
- Format: MP4 (H.264) for direct share, GIF fallback for low-bandwidth
- Resolution: 720p (mobile-optimized)
- Overlay: Player name + dynasty + "IMPERIO INMOBILIARIO" watermark + deep link
- Export: One-tap share to WhatsApp, Instagram Stories, TikTok, Copy Link

**Implementation:** Client records a rolling 30s replay buffer. When a clip-worthy moment occurs, a "Compartir clip" button appears for 5s. Tap generates the clip from the buffer.

**Phase 1 (MVP):** Clip markers only (manual screenshot encouraged). Share card with moment description.
**Phase 2:** Full video clip export.

### 3.5.10 Anonymous Spectating (NEW in v12.1)

Public matches can be spectated without an account:

```
1. Non-player visits imperio.game/watch/{roomId} or browses public matches
2. Enters as anonymous Watcher (no Mood vote, no Rumor, no betting)
3. Can view full match state + Quick Chat + events
4. After match ends: CTA "Crea tu cuenta para jugar" (one-tap signup)
5. If signup: receives GD 10,000 welcome bonus + redirect to tutorial
```

**Watcher Progression** (for registered Watchers):
- +1 Watcher XP per correct auction bet prediction
- +2 Watcher XP per match watched to completion
- Thresholds: 10 XP = "Observador" badge, 50 XP = "Analista" badge, 100 XP = "Experto" badge
- Badges visible on profile when player becomes an Owner
```

---

# AMENDMENT H6: DEAL PRICE VALIDATION

**Action:** [INSERT AFTER] Section 7.2.2 (line 838), before 7.2.3

```markdown
### 7.2.2b Deal Price Validation (NEW in v12.1)

Server validates all deals to prevent kingmaking via asset dumping:

| Template | Validation Rule |
|----------|----------------|
| Cash for Property (#1) | `amount >= BasePrice × 0.6 AND amount <= BasePrice × 2.0` |
| Property Swap (#2) | `abs(BasePrice_A - BasePrice_B) <= max(BasePrice_A, BasePrice_B) × 0.5` |
| Sell Package (#7) | `amount >= sum(BasePrice of properties in package) × 0.6` |
| Election Pact (#4) | `payment >= GD 20,000 AND payment <= GD 150,000` |
| Non-compete (#5) | `payment >= GD 10,000 AND payment <= GD 100,000` |
| Security Escort (#6) | `payment >= GD 15,000 AND payment <= GD 80,000` |

**Rejection message:** "Oferta rechazada por el regulador: precio fuera de rango permitido."

**Design rationale:** Template-only deals prevent free-form abuse, but without price floors a losing player can dump GD 2M in assets to a friend for GD 1. The 0.6x floor allows discounted sales (fire sale, urgent cash need) while preventing zero-value transfers. The 2.0x ceiling prevents money laundering (overpaying for a cheap property to transfer cash).
```

---

# AMENDMENT H7: DYNASTY SOCIAL STRUCTURE

**Action:** [REPLACE] Section 3.4 (lines 265-291)

```markdown
## 3.4 Dynasties

Dynasties are social groups for prestige rivalry and collective competition. Defined in `data/teams.json` (Phase 1: pre-defined sets) and player-created (Phase 2).

### 3.4.1 Team Sets (Phase 1)

| Set | Count | Use Case |
|-----|-------|----------|
| PARODY | 17 | Public rooms (safer, satirical) |
| REAL_LABEL | 17 | Friends rooms (with disclaimer) |

### 3.4.2 PARODY Set (17)
Brecha, Romerazo, Gloriaz, Intercopia, Hochi, Benavidios, Arias V, Navarro G, Marsanito, Lindlitas, Fishmanazo, Acunazo, Dayer, Mulderon, Mattita, Ananos, Ananos J

### 3.4.3 REAL_LABEL Set (17)
Brescia, Romero, Rodriguez, Intercorp, Hochschild, Benavides, Arias Vargas, Navarro Grau, Marsano, Lindley, Fishman, Acuna, Dyer, Mulder, Matta, Ananos, Ananos Jeri

### 3.4.4 Mandatory Disclaimer (Spanish)
> "Los nombres de dinastias se usan solo como fantasia/satira. No hay afiliacion ni representacion de personas u organizaciones reales."

**Display locations**: Lobby, share pages, team selection screen

### 3.4.5 Team Selection
- Pick during profile setup, OR
- Auto-assign randomly on first join (change once pre-match)
- Room option: `Unique Teams` (if ON, each team used by only one Owner)

### 3.4.6 Dynasty Social Features (NEW in v12.1)

**Dynasty Member List:**
- Visible in Dynasty tab: all members with name, league badge, lifetime GD
- Sorted by lifetime GD (descending)

**Dynasty Quick Chat** (between matches):
- Same 12 preset phrases as in-match Quick Chat
- Available in lobby when viewing dynasty tab
- Rate limit: 5 messages per minute per player
- Messages visible to all dynasty members currently online

**Dynasty Invite Link:**
- Each dynasty has a shareable link: `imperio.game/dynasty/{dynastyId}`
- Link shows dynasty name, member count, aggregate GD, league badge
- "Unirse" button joins the dynasty (if roster not full)

**Dynasty Roster:**
- Phase 1: Max 50 members per dynasty (pre-defined sets have unlimited)
- Phase 2: Player-created dynasties, max 50 members, creation cost GD 100,000 from persistent wallet

### 3.4.7 Dynasty Collective Goals (NEW in v12.1)

See Section 3.5.6 (Weekly Dynasty Challenge) for the collective challenge system.

Dynasty aggregate GD leaderboard is the primary competitive layer between dynasties. Members contribute their lifetime GD to the dynasty total. This creates recruitment pressure: invite strong players to climb the dynasty ladder.
```

---

# AMENDMENT H8: PORTADA VIRAL UPGRADE

**Action:** [REPLACE] Appendix D1 (lines 2892-2927)

```markdown
## D1: Portada (End-of-Match Share Card) — REVISED in v12.1

**Generated for:** Every Owner in the match (not just the winner).

**Dimensions**: 1200×630 (landscape) + 1080×1920 (stories)

**Layout (landscape)**:
```
+------------------------------------------------------------+
|  IMPERIO INMOBILIARIO                        [QR Code]      |
|  ============================                               |
|                                                             |
|  +------------+                                             |
|  |  PLAYER    |   {PLAYER_NAME}                             |
|  |  PORTRAIT  |   Dinastia {TEAM}                           |
|  |            |   #{RANK} de {TOTAL} -- Riqueza: GD {WEALTH}|
|  +------------+   Props: {COUNT} | Tratos: {DEALS}          |
|                    Mejor jugada: {HIGHLIGHT}                 |
|                                                             |
|  -----------------------------------------------------------+
|  "TITULAR 1 DEL MATCH"                                      |
|  "TITULAR 2 DEL MATCH"                                      |
|                                                             |
|  [Sticker 1]  [Sticker 2]                                   |
|                                                             |
|  PUEDES GANARME? -> imperio.game/play/{matchId}             |
|  -----------------------------------------------------------+
|  Los nombres de dinastias se usan solo como fantasia/       |
|  satira. No hay afiliacion con personas reales.             |
+------------------------------------------------------------+
```

**Per-player customization:**

| Element | Winner (#1) | Other Players |
|---------|-------------|---------------|
| Border | Gold metallic | Silver (top 3), Bronze (4+) |
| Rank display | "EL MAGNATE" | "#3 de 8" |
| Stickers | Victory stickers | Consolation/funny stickers |
| Tone | Triumphant | Aspirational ("La proxima es mia") |

**Required Elements**:
- Masthead "IMPERIO INMOBILIARIO"
- Player portrait + dynasty crest + color
- Rank, final wealth, property count, deals completed
- Personal highlight moment (biggest deal, best auction, closest bid)
- 2-3 tabloid headlines from the match
- Challenge CTA: "Puedes ganarme?" with deep link
- QR code + short link (`imperio.game/play/{matchId}`)
- League badge (current tier)
- **Mandatory disclaimer footer (Spanish)**

**Share Triggers** (each generates appropriate share card):

| Trigger | Card Type | Auto-Generated? |
|---------|-----------|-----------------|
| Match end | Portada (per player) | Yes |
| Mid-match achievement | Trophy card (D2) | Yes |
| League promotion | League Card | Yes |
| 3+ win streak | Streak Card | Yes |
| Dynasty enters top 10 weekly | Dynasty Card | Yes |
| Personal best Wealth | Record Card | Yes |

**Share Integration:**
- WhatsApp (primary — LATAM market)
- Instagram Stories (1080×1920 format)
- Telegram
- Copy Link
- "Guardar imagen" (save to device)
```

---

# AMENDMENT H9: RECONNECTION FLOW

**Action:** [REPLACE] Section 12.6 (lines 1371-1378)

```markdown
## 12.6 Reconnection Handling (REVISED in v12.1)

### 12.6.1 Disconnect Timeline

| Time Since Disconnect | Behavior |
|-----------------------|----------|
| 0-10s | Grace period. Player's actions timeout normally (no bid = no bid, no action = Pass). No bot activation. |
| 10s-60s | **Bot activates.** Bot heuristics (A8) take over all decisions. Match continues normally. Player slot reserved. |
| 60s-5min | Bot continues. Reconnection still allowed. Player receives push notification: "Tu partida continua. Reconecta ahora." |
| >5min | Bot continues permanently. Player can rejoin at any point during the remaining match. Seat is never locked. |

### 12.6.2 Reconnection Protocol

When a disconnected player reconnects:

```
1. Client sends RECONNECT { sessionId, matchId }
2. Server validates session
3. Server sends STATE_SNAPSHOT (full current state)
4. Client rebuilds UI from snapshot
5. Server deactivates bot, returns control to player
6. If current phase has active timer: player joins mid-phase with remaining time
7. If current phase has already collected this player's input (bot acted):
   player resumes at NEXT phase
```

**Queued actions:** Any actions the client queued during disconnect are discarded. The bot's decisions during disconnect are final and cannot be reversed.

### 12.6.3 Push Notifications (Phase 2)

| Event | Notification |
|-------|-------------|
| Disconnect during active match | "Tu partida sigue. Reconecta: imperio.game/rejoin/{matchId}" (after 30s) |
| Match about to end (last round) | "Ultima ronda! Vuelve ahora." |
| Dynasty member online | "Tu compañero {name} esta jugando." (if opted in) |

### 12.6.4 Rate Limiting

Max 5 reconnection attempts per minute per player. Exceeding this triggers a 60s cooldown before next attempt.
```

---

# AMENDMENT H10: BOT HEURISTICS EXPANSION

**Action:** [INSERT AFTER] Section A8 H5 (line 1983), before A9

```markdown
### H6: Mood/Vote Logic (NEW in v12.1)

```typescript
function decideMoodVote(bot: OwnerState, options: MoodOption[]): string {
  // Prefer the positive/beneficial option
  const positive = options.find(o => o.sentiment === "positive");
  if (positive) return positive.id;
  // Fallback: pick the option that affects the most properties the bot doesn't own
  return options[0].id;
}

function decideElectionVote(bot: OwnerState, candidates: Candidate[]): string {
  // Vote for the candidate whose bonus helps bot's tag portfolio
  const tagCounts = countBotPropertyTags(bot);
  return candidates.reduce((best, c) =>
    (tagCounts[c.bonusTag] || 0) > (tagCounts[best.bonusTag] || 0) ? c : best
  ).id;
}
```

### H7: Package Logic (NEW in v12.1)

```typescript
function shouldCreatePackage(bot: OwnerState): PropertyPackage | null {
  if (bot.cash < 40_000) return null;

  // Check each package category
  for (const category of ["RESIDENTIAL_BUNDLE", "LOGISTICS_HUB", "COMMERCIAL_STRIP"]) {
    const eligible = getEligibleProperties(bot, category);
    if (eligible.length >= 3) {
      return { category, propertyIds: eligible.slice(0, 3) };
    }
  }
  return null;
}
```

### H8: Fame Spend Logic (NEW in v12.1)

```typescript
function decideFameSpend(bot: OwnerState, event: EventResult): FameSpendUsed | null {
  // Control de danos: spend 2 Fame to halve cash loss if loss >= 100K
  if (event.cashLoss >= 100_000 && bot.fame >= 2) {
    return "CONTROL_DE_DANOS";
  }
  // Cortina de humo: spend 3 Fame to cancel Fame loss if loss >= 2
  if (event.fameLoss >= 2 && bot.fame >= 3) {
    return "CORTINA_DE_HUMO";
  }
  return null;
}
```

### H9: Deal Evaluation (NEW in v12.1)

```typescript
function evaluateDeal(bot: OwnerState, deal: DealOffer): "ACCEPT" | "REJECT" {
  switch (deal.template) {
    case "CASH_FOR_PROPERTY":
      // Accept if offered >= 1.1x BasePrice (good deal)
      const property = getProperty(deal.propertyId);
      return deal.amount >= property.basePrice * 1.1 ? "ACCEPT" : "REJECT";

    case "BANK_LOAN":
      // Accept if cash < 80K and no active loan
      return bot.cash < 80_000 && !bot.hasActiveLoan ? "ACCEPT" : "REJECT";

    default:
      return "REJECT";  // Conservative on complex deals
  }
}
```

### H-RESERVE: Standardized Reserve (FIX in v12.1)

All bot heuristics use a unified GD 100,000 reserve:

```typescript
const BOT_CASH_RESERVE = 100_000;

// H1 shouldPass: bot.cash < BOT_CASH_RESERVE
// H1 maxBid: bot.cash - BOT_CASH_RESERVE
// H2 calculateBid: Math.min(valuation, bot.cash - BOT_CASH_RESERVE)
// H5 selectAction: bot.cash < BOT_CASH_RESERVE → PASS
```

The previous inconsistency between GD 100K (H1) and GD 50K (H2) is resolved. All functions use `BOT_CASH_RESERVE`.
```

---

# SUMMARY OF ALL AMENDMENTS

| ID | Section | Change Type | Lines Affected |
|----|---------|-------------|---------------|
| C1 | New Part 14 | INSERT | After line 1491 |
| C2 | Section 3.5 | REPLACE | Lines 292-327 |
| H1 | Section 6.5 | REPLACE | Lines 777-785 |
| H2 | Section 6.6 | REPLACE | Lines 787-792 |
| H3 | New 6.8 | INSERT | After line 798 |
| H4 | Section 8.4 | REPLACE | Lines 1072-1089 |
| H5 | New 3.5.8-3.5.10 | INSERT | After new 3.5.7 |
| H6 | New 7.2.2b | INSERT | After line 838 |
| H7 | Section 3.4 | REPLACE | Lines 265-291 |
| H8 | Appendix D1 | REPLACE | Lines 2892-2927 |
| H9 | Section 12.6 | REPLACE | Lines 1371-1378 |
| H10 | Appendix A8 | INSERT | After line 1983 |

**Version note:** Applying all amendments advances the spec to v12.1. Update the header (line 1) and add a v12.0 → v12.1 changelog entry.

---

*End of Spec Amendments v12.1*
