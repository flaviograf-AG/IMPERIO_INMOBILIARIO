# IMPERIO INMOBILIARIO — Unified Master Specification v12

**Version:** 12.0
**Date:** 2026-02-04
**Status:** Golden Master (v11 + Graf Dollar Economy + Weakness Fixes)

---

## CHANGELOG (v11 → v12)

### Economy Overhaul
- **Graf Dollar (GD) currency**: All in-match values scaled to tycoon-realistic amounts (GD 200K-2M properties)
- **Persistent GD wallet**: Match final Wealth → lifetime leaderboard ranking (prestige-only, no in-match advantage)
- **Leaderboard system**: Tiered leagues (Inquilino → Emperador), seasonal decay, dynasty rankings
- **Formula recalibration**: All formulas updated to GD scale (Wealth, Rent, OpsCost, costs)

### Content Expansion
- **Property deck**: Expanded from 13 to 25 properties with portfolio-concept theming
- **Property Packaging**: New "Paquete Inmobiliario" mechanic — group 3+ properties for synergy bonuses
- **MOOD phase**: Full specification with 8 mood options, Owner fallback when 0 Watchers, tie-breaking rules
- **Quick Chat**: 12 preset Spanish phrases for in-match communication

### Gameplay Fixes
- **Major/Minor action split**: Pick 1 Major + 0-1 Minor per round (doubles decision space)
- **Quick Mode Mini-Election**: R5 mini-election with 2 candidates
- **R1 Consolation Rule**: GD 50,000 subsidy for Owners with 0 properties after R1
- **Smart Deal Suggestions**: Server-generated contextual deal offers
- **Deal template #7**: SELL_PACKAGE for selling entire property packages
- **Sound fix**: sfx_cash_loss changed to orchestral sting (matches Tycoon Luxury aesthetic)

### New Data Files
- `data/properties.json` — 25 properties with GD values and packageCategory
- `data/moods.json` — 8 mood options with effects
- `data/quickchat.json` — 12 preset Spanish phrases

### New Spec Sections
- **Part 13**: Graf Dollar Metagame Economy (persistent wallet, leaderboard, leagues, seasonal decay)

### Previous Changelogs

<details>
<summary>v10 → v11</summary>

#### Merged from v10
- **Architecture**: Satellite Data strategy (external JSON in `/data/`)
- **UX**: Visual Ledger phase (`VISUAL_LEDGER`) between INCOME and ACTION
- **UI**: Deal Tray (non-blocking) + "Modo Ocupado" (Busy Mode)
- **Security**: JIT round seed with `serverSecret`
- **Network**: Delta packets (`OP_DELTA`)
- **AI**: Bot heuristics for Tutorial/Autopilot (Appendix A8)

#### New in v11
- **Open Source Foundation Matrix** with repository references
- **Viral Gaming Best Practices** (2025-2026 research)
- **Complete 52+ Event Deck** with Spanish headlines
- **Colyseus Integration Examples** (TypeScript/GDScript)
- **Referral Mechanics** documentation
- **Watcher Betting** on auctions
- **Phase Timing Diagrams**

</details>

---

# PART 1: EXECUTIVE SUMMARY & DESIGN PHILOSOPHY

## 1.1 Tagline
> **Become the richest and most famous landlord dynasty in Lima—while the city, the press, and Public Mood try to take you down.**

## 1.2 Design Goals

| Goal | Implementation |
|------|----------------|
| **Fast to learn (≤60s)** | Each round: one Bid + one Action |
| **High virality** | Public rooms, Watchers influence outcomes, shareable Portada + Logros |
| **Low infra + low hardware** | Turn-based, 2D UI-first, Godot Web export, authoritative server |
| **Tycoon feel without complexity** | Demand cycles + portfolio overhead + upgrades |
| **Strong Lima/Peru flavor** | Spanish-only player content, last-10-years macro themes |

## 1.3 Language Rules (Hard)
- All **player-facing UI, cards, headlines, and prompts**: Spanish
- All **logic variables and code comments**: English
- Spanish strings loaded from `data/locales_es.json`

## 1.4 Hard Constraints (Must Not Change)

| Constraint | Value |
|------------|-------|
| Max Owners | 12 (Standard mode) |
| Max Participants | 20 (Owners + Watchers) |
| Watchers | Unlimited within room cap |
| Events | Targeted at single Owner (probability scales with properties/wealth/fame) |
| Insurance | Has cost AND benefit |
| Catch-up mechanics | Mandatory |
| Deals | Template-only: 7 types, no free-form |
| Graf Design System | Admin UI only |
| Public rooms | Viewable; profile shares unlisted by default |
| Currency | Graf Dollar (GD) — prestige leaderboard currency; no in-match advantage from persistent wallet |
| Match fairness | All players start with identical GD per mode; persistent wallet has zero in-match effect |

## 1.5 Open Source Foundation Strategy

This project leverages proven open-source foundations to minimize development risk and maximize code quality. See **Appendix O** for the complete evaluation matrix.

**Primary Stack:**
- **Server**: Colyseus (authoritative state, rooms, WebSocket)
- **Client**: Godot 4.x with optional GodotJS for TypeScript
- **Database**: PostgreSQL
- **Cache**: Redis (optional, for rate limiting)

---

# PART 2: GAME MODES & CONFIGURATIONS

## 2.1 Mode Parameters

All modes share rules; only parameters differ. Configuration loaded from `data/modes.json`.

### 2.1.1 Quick Mode (Viral Default)
```yaml
mode: QUICK
owners: 4-12
rounds: 7
timers:
  bid: 25s
  action: 25s
marketRowSize: 5
targetedEventSlots: 1 (+1 if Owners ≥ 9)
globalShockCap: 1
elections: [R5]  # Mini-Election (2 candidates, simplified)
specialRounds:
  R3: Guerra de Distrito
  R5: Mini-Eleccion
  R6: Ventana Bancaria
```

### 2.1.2 Standard Mode
```yaml
mode: STANDARD
owners: 4-12
rounds: 10
timers:
  bid: 35s
  action: 35s
marketRowSize: 6
targetedEventSlots: 1 (+1 if Owners ≥ 7) (+1 if Owners ≥ 11)
globalShockCap: 2
elections: [R3, R7]
specialRounds:
  R3: Guerra de Distrito
  R5: Ventana Bancaria
  R9: Subasta de Prestigio
```

### 2.1.3 Long Mode (Strategic)
```yaml
mode: LONG
owners: 4-8
rounds: 14
timers:
  bid: 40s
  action: 40s
marketRowSize: 6
targetedEventSlots: 2 (+1 if Owners ≥ 7)
globalShockCap: 2
elections: [R4, R10]
specialRounds:
  R4: Guerra de Distrito
  R8: Ventana Bancaria
  R13: Subasta de Prestigio
```

## 2.2 Room Types

| Type | Join Method | Listed | Default Team Set |
|------|-------------|--------|------------------|
| PUBLIC | Open join | Yes (lobby) | PARODY |
| FRIENDS | Token/QR | No | REAL_LABEL |

**Host Controls:**
- Lock seats after Round 1
- Override team set (with disclaimer acceptance for REAL_LABEL in public)
- Enable/disable "Unique Teams" (each team used by only one Owner)

## 2.3 Starting Values by Mode

All players in a match start with identical resources — no persistent-wallet influence.

| Resource | Quick | Standard | Long |
|----------|-------|----------|------|
| Cash | GD 500,000 | GD 650,000 | GD 800,000 |
| Debt | 0 | 0 | 0 |
| Fame | 0 | 0 | 0 |
| Compliance | 1 | 1 | 1 |
| Political Exposure | 0 | 0 | 0 |
| Debt Cap | GD 750,000 | GD 750,000 | GD 750,000 |

**What 500K buys in Quick:** 2 entry properties (200K+250K=450K, GD 50K left) OR 1 mid-range (500K, GD 0 left). Players must choose between breadth (multiple cheap) or concentration (one strong asset).

**Game arc:** R1-R2 buy entry-level (200-400K). R3-R5 accumulate rent, bid on mid/upper tier. R6-R7 premium properties are aspirational targets requiring loans + accumulated wealth.

---

# PART 3: ROLES, IDENTITY & VIRAL SYSTEMS

## 3.1 Room Capacity
- Up to **20 participants** per room
- Maximum **12 Owners** at once
- Remaining participants are **Watchers**

## 3.2 Owners

**Per-Round Actions:**
- One bid (on one property)
- One Major action (Develop/Package/Deal/Pass)
- One Minor action, optional (Insure/Security/PR/Compliance Audit)
- One deal offer (outgoing)

**Per-Match Limits:**
- One "Romper Pacto" (break pact) — consumes both Major and Minor slots
- One "Yape de emergencia" (Fame→Cash conversion)

### 3.2.1 Quick Chat ("Mensajes Rapidos")

12 preset Spanish phrases for in-match communication. Defined in `data/quickchat.json`.

| ID | Phrase |
|----|--------|
| QC-01 | Quiero hacer un trato |
| QC-02 | No me interesa |
| QC-03 | Vamos juntos |
| QC-04 | Ojo con ese |
| QC-05 | Buena jugada |
| QC-06 | Estoy en la ruina |
| QC-07 | Eso no vale |
| QC-08 | Vamos a ganar |
| QC-09 | Socorro |
| QC-10 | No me toquen |
| QC-11 | Hagamos negocio en {DISTRITO} |
| QC-12 | jajaja |

**Rate limit:** 2 messages per phase per Owner.
**Display:** Speech bubble above avatar, fades after 3s.
**Protocol:** Client sends `SUBMIT_QUICK_CHAT` with `{ phraseId, districtParam? }`. Server broadcasts `QUICK_CHAT_MSG` to all clients.

## 3.3 Watchers

**Influence Mechanics:**
- One tap per round for Mood vote
- Occasional award votes
- Optional reaction emojis (no chat required)

### 3.3.1 Watcher "Rumor" Button
- **Limit**: Once per match per Watcher
- **Effect**: Target Owner receives `+0.2 RiskScore` next round
- **Server Cap**: Total Rumor influence per Owner per round capped at `+0.6`
- **UI Label**: "Correr el rumor"

### 3.3.2 Watcher Auction Betting (NEW in v11)
- Watchers can bet on which Owner will win each auction
- **Reward**: Correct prediction grants +1 Watcher Fame
- **Limit**: One bet per auction per Watcher
- **UI**: Quick-select Owner chips during BID phase

## 3.4 Teams (Dynasties)

Teams are fantasy labels for prestige rivalry. Defined in `data/teams.json`.

### 3.4.1 Team Sets

| Set | Count | Use Case |
|-----|-------|----------|
| PARODY | 17 | Public rooms (safer, satirical) |
| REAL_LABEL | 17 | Friends rooms (with disclaimer) |

### 3.4.2 PARODY Set (17)
Brecha, Romerazo, Gloriaz, Intercopia, Hochi, Benavidios, Arias V, Navarro G, Marsanito, Lindlitas, Fishmanazo, Acunazo, Dayer, Mulderon, Mattita, Ananos, Ananos J

### 3.4.3 REAL_LABEL Set (17)
Brescia, Romero, Rodríguez, Intercorp, Hochschild, Benavides, Arias Vargas, Navarro Grau, Marsano, Lindley, Fishman, Acuña, Dyer, Mulder, Matta, Añaños, Añaños Jerí

### 3.4.4 Mandatory Disclaimer (Spanish)
> "Los nombres de dinastías se usan solo como fantasía/sátira. No hay afiliación ni representación de personas u organizaciones reales."

**Display locations**: Lobby, share pages, team selection screen

### 3.4.5 Team Selection
- Pick during profile setup, OR
- Auto-assign randomly on first join (change once pre-match)
- Room option: `Unique Teams` (if ON, each team used by only one Owner)

## 3.5 Viral & Retention Hooks

### 3.5.1 Share Assets
| Asset | Dimensions | Trigger |
|-------|------------|---------|
| Portada | 1200×630 + 1080×1920 | Match end |
| Trophy (Logro) | 1080×1920 | Mid-match achievement |

### 3.5.2 Competitive Systems
- **Dynasty Ladder**: Weekly seasons with leaderboard
- **Revancha al toque**: Quick rematch flow

### 3.5.3 Watcher Awards
- "Favorito del público" (voted by Watchers)
- "Ampay del día" (scandal highlight)

### 3.5.4 Referral System (NEW in v11)
- **Invite Link**: Each player gets unique referral code
- **Referrer Reward**: +50 Dynasty XP per referred player who completes Tutorial
- **Referee Reward**: +1 Cosmetic Badge ("Recomendado")
- **Tracking**: Server records `referredBy` on account creation

### 3.5.5 Streak System (NEW in v11)
- **Daily Login**: +10 Dynasty XP base
- **Streak Multiplier**: Day 2 = 1.5x, Day 3 = 2x, Day 7+ = 3x
- **Streak Break**: Resets to Day 1 after 48h without login

### 3.5.6 Leaderboard System (NEW in v12)

Lifetime GD leaderboard with tiered leagues. See **Part 13** for full specification.

- **5 league tiers**: Inquilino → Propietario → Inversionista → Magnate → Emperador
- **Monthly seasonal decay**: balances > GD 5M decay 10% monthly
- **Dynasty leaderboard**: aggregate team GD (strongest viral hook — recruit friends)
- **Contextual views**: Friends, Regional (Lima districts), Dynasty, All-time

---

# PART 4: ONBOARDING & TIME-TO-FUN

## 4.1 Design Principles (Viral Best Practices 2025-2026)

| Principle | Implementation |
|-----------|----------------|
| Interactive tutorials, not text walls | Coach overlays with ≤2 Spanish sentences |
| Core gameplay within 60 seconds | Tutorial forces first auction participation |
| Milestone-based feature unlocking | Achievements gate advanced features |
| First-win guarantee | Bot heuristics ensure user wins R1 auction |

## 4.2 Mandatory First-Time Tutorial Match

**Trigger**: First time an account joins any room

**Format:**
```yaml
rounds: 2
owners: 3 minimum (fill with Bots)
setup: 1 Human vs 2-3 Bots
```

**Scripted Outcomes:**
- **Round 1**: User wins a cheap auction with suggested bid (Bot heuristics: bid GD 50,000)
- **Round 2**: User targeted by mild event; sees insurance mitigate it

**Constraints:**
- Coach overlays: max 2 Spanish sentences per phase
- Skippable after completing once

**Reward:**
- Achievement: `A-TUT-01 "Primer Casero"`
- Cosmetic badge

## 4.3 Bot Heuristics for Tutorial

See **Appendix A8** for complete bot logic. Key Tutorial behaviors:

| Round | Bot Behavior |
|-------|--------------|
| R1 | Always bid GD 50,000 (ensures human wins) |
| R2 | Do not mitigate event (shows insurance value) |

---

# PART 5: CORE LOOP & STATE MACHINE

## 5.1 Phase Sequence (Authoritative)

### 5.1.1 Normal Rounds (11 Phases)
```
REVEAL → BID → INCOME_CALC → VISUAL_LEDGER → ACTION → MOOD → DEALS → EVENTS → PRESS → [AWARDS] → SNAPSHOT
```

### 5.1.2 Election Rounds (VOTE replaces MOOD)
```
REVEAL → BID → INCOME_CALC → VISUAL_LEDGER → ACTION → VOTE → DEALS → EVENTS → PRESS → [AWARDS] → SNAPSHOT
```

## 5.2 Phase Timing Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│ REVEAL (2s)                                                             │
│   Server broadcasts: roundSeed, Market Row, Signals, Mood Options       │
├─────────────────────────────────────────────────────────────────────────┤
│ BID (25-40s based on mode)                                              │
│   Players submit sealed bids simultaneously                             │
│   Watchers place auction bets                                           │
├─────────────────────────────────────────────────────────────────────────┤
│ INCOME_CALC (instant)                                                   │
│   Server calculates: Rent, OpsCost, Debt, LeaderTax, ForcedSales       │
├─────────────────────────────────────────────────────────────────────────┤
│ VISUAL_LEDGER (4.0s) ◄── NEW IN v10                                     │
│   CLIENT BLOCKING: Action inputs locked                                 │
│   UI animates Income Receipt                                            │
├─────────────────────────────────────────────────────────────────────────┤
│ ACTION (25-40s based on mode)                                           │
│   Major: Develop/Package/Deal/Pass + Minor (opt): Insure/Security/PR/Audit │
├─────────────────────────────────────────────────────────────────────────┤
│ MOOD or VOTE (15s; 10s if 0 Watchers)                                   │
│   Watchers (or Owners if 0 Watchers) vote on Mood modifier OR Election  │
├─────────────────────────────────────────────────────────────────────────┤
│ DEALS (instant)                                                         │
│   Resolve accepted deals deterministically                              │
├─────────────────────────────────────────────────────────────────────────┤
│ EVENTS (2s per event)                                                   │
│   Select targets, apply effects, apply fairness caps                    │
├─────────────────────────────────────────────────────────────────────────┤
│ PRESS (2s)                                                              │
│   Generate 1-3 Spanish headlines                                        │
├─────────────────────────────────────────────────────────────────────────┤
│ AWARDS (optional, 10s)                                                  │
│   Watcher voting for mid-match awards                                   │
├─────────────────────────────────────────────────────────────────────────┤
│ SNAPSHOT (instant)                                                      │
│   Emit end-of-round state to all clients                                │
└─────────────────────────────────────────────────────────────────────────┘
```

## 5.3 Hard Ordering Rule (No Ambiguity)

1. Resolve bids (determine auction winners)
2. Apply Income (rent + signals + ops + debt + scheduled + leader tax)
3. **Visual Ledger animation** (client-side, server waits 4.0s)
4. Collect actions
5. Resolve Mood/Vote result (queued effects)
6. Resolve deals at DEALS phase (post-action ownership authoritative)
7. Select targets and resolve events (apply caps)
8. Generate headlines
9. Emit achievements
10. Emit snapshots

**Timeouts:**
- No bid → No bid submitted
- No action → Pass
- No mood/vote → No vote counted

Server proceeds regardless of client response.

## 5.4 REVEAL Phase

Server reveals:
- **Market Row**: Properties available for auction
- **Market Signals**: Two signals affecting rent by tag
- **Mood Options**: Two options (non-election rounds)
- **Special Round Rule**: If applicable

### 5.4.1 Round Seed Security (v10 Fix)

**Vulnerability Fixed**: Sending `matchSeed` at start allowed clients to predict all future coin flips.

**Solution**:
```
serverSecret = generateSecureRandom()  // Never sent to client
roundSeed = sha256(matchSeed + serverSecret + roundNumber)
```

**Constraint**: Server sends `roundSeed` ONLY in the REVEAL packet for that specific round.

## 5.5 BID Phase (Sealed, Simultaneous)

Each Owner may bid on **one** property:
- `(propertyId, amount)`
- Optional: spend 1 Fame as tie-break token (max 1 per round)

### 5.5.1 Payment Model (Winner-Only)
- Only winning Owner pays bid amount
- Losing Owners pay nothing (no escrow, no refunds)

### 5.5.2 Tie-Break Order
1. Higher bid amount
2. Fame token used (yes > no)
3. Higher current Fame
4. **Deterministic seeded coin flip**

### 5.5.3 Deterministic Coin Flip (Hash)
```typescript
function tieBreak(roundSeed: string, propertyId: string, tiedOwnerIds: string[]): string {
  const sorted = tiedOwnerIds.slice().sort();
  const input = `${roundSeed}|${propertyId}|${sorted.join(',')}`;
  const hash = sha256(input);
  const index = parseInt(hash.slice(0, 8), 16) % sorted.length;
  return sorted[index];
}
```

## 5.6 INCOME_CALC Phase (Server-Side, Instant)

Server calculates in order:
1. Rent income (with Market Signals) — GD amounts per property
2. Operating costs (portfolio overhead) — GD 8,000 per unit
3. Debt service
4. Leader friction tax ("Impuesto a la fama") — GD 8,000/15,000
5. Scheduled effects (loan repayments, etc.)
6. Forced sales if cash < 0

### 5.6.1 R1 Consolation Rule ("Subsidio de arranque")

If an Owner owns **0 properties** at end of R1 INCOME_CALC, grant **GD 50,000**.

- One-time, R1 only
- Headline: *"{OWNER} recibe subsidio municipal para arrancar."*
- Rationale: 50K = 10% of starting cash (500K Quick). Enough to cover insurance or development costs but not enough to close the gap on someone who bought a property.

## 5.7 VISUAL_LEDGER Phase (NEW in v10)

**Purpose**: Visualize the "Invisible Accountant" calculations

**Duration**: 4.0 seconds (server-enforced)

**Client Behavior**:
- Lock ACTION input buttons
- Display animated Income Receipt:
  ```
  ┌──────────────────────────────┐
  │  Rentas:    +GD 95,000  🟢   │
  │  Costos:    -GD 16,000  🔴   │
  │  ──────────────────────      │
  │  Total:     +GD 79,000       │
  │  [Cash counter animates]     │
  └──────────────────────────────┘
  ```

**Constraint**: Players CANNOT click Action buttons until animation completes.

## 5.8 ACTION Phase (Major/Minor Split)

Choose **one Major action** plus **zero or one Minor action**:

### Major Actions (pick 1)

| Action | Cost | Effect |
|--------|------|--------|
| Develop | GD 60,000-100,000 (meter-dependent) | +GD 12,000 rent permanently, +GD 40,000 worth |
| Package | GD 40,000 | Create "Paquete Inmobiliario" from 3+ owned properties (see 7.9) |
| Deal (offer) | 0 | Propose template deal |
| Pass | 0 | Do nothing |

### Minor Actions (pick 0 or 1)

| Action | Cost | Effect |
|--------|------|--------|
| Insure | GD 20,000-60,000 | Add/upgrade insurance |
| Security Contract | GD 20,000 | Protection until end of next round |
| PR Push | 0 | +1 Fame, +0.5 RiskScore next round |
| Compliance Audit | 0 | +1 Compliance, -0.5 RiskScore next round |

### Special: Romper Pacto
- Cost: 0 (consumes both Major AND Minor slots)
- Effect: Cancel active pact (once per match)

This Major/Minor split doubles decision space without adding time: you can now Develop AND Insure, or Deal AND PR Push in the same round.

## 5.9 MOOD Phase (Non-Election Rounds)

### 5.9.1 Mood Option Generation

Server picks **2 mood options** per round from `data/moods.json` (8 total). Selection rules:
- One positive/beneficial, one negative/disruptive
- No repeats in consecutive rounds

### 5.9.2 Mood Options

| ID | Name | Effect |
|----|------|--------|
| M-01 | Boom de alquileres | All rent +GD 10,000 next round |
| M-02 | Miedo al mercado | All rent -GD 10,000 next round |
| M-03 | Fiebre inversora | Develop cost -GD 30,000 next round |
| M-04 | Vecinos molestos | All RiskScore +0.3 next round |
| M-05 | Calma chicha | Suppress 1 event next round |
| M-06 | Prensa hambrienta | Fame gains/losses ×2 next round |
| M-07 | Ojo fiscalizador | High-compliance owners (≥2) get +GD 20,000 bonus next round |
| M-08 | Gentrificacion express | Properties under GD 400,000 get +GD 12,000 rent next round |

### 5.9.3 Voting

**When Watchers present (≥1):** Watchers vote between 2 options (one tap, 15s timer).
- Winning modifier queued to apply next round
- Optional: show live percentages (host option)

**When 0 Watchers (Owner fallback):** Owners vote on mood options instead.
- Timer reduced to **10s**
- Same voting rules apply
- Each Owner gets one vote

### 5.9.4 Tie-Breaking

1. Higher vote count wins
2. If tied: positive-delta option wins (the one that benefits players)
3. If still tied: deterministic coin flip using `roundSeed`

### 5.9.5 MoodOption Schema

```typescript
interface MoodOption {
  id: string;           // "M-01"
  nombre_es: string;    // "Boom de alquileres"
  effect: {
    type: "RENT_ALL" | "DEVELOP_COST" | "RISK_ALL" | "SUPPRESS_EVENT" | "FAME_MULTIPLIER" | "COMPLIANCE_BONUS" | "RENT_TIER";
    delta?: number;
    multiplier?: number;
    threshold?: number;  // for RENT_TIER: basePrice threshold
    duration: number;    // rounds
  };
  sentiment: "POSITIVE" | "NEGATIVE" | "NEUTRAL";
}
```

## 5.10 VOTE Phase (Election Rounds)

**Owners choose:**
- Candidate A or B (or C in Standard/Long)
- Contribution: GD 0 / GD 25,000 / GD 50,000 (counts as "support points")

**Watchers vote:**
- Candidate A or B (one tap)

### 5.10.1 Election Winner Determination

```
ScoreA = Σ(ownerContributionToA)
ScoreB = Σ(ownerContributionToB)

if ScoreA != ScoreB:
    winner = higher score
elif watcherVoteA != watcherVoteB:
    winner = higher watcher vote
else:
    winner = deterministic coin flip
```

**Effects:**
- Winner bundle applies for **next 2 rounds** (meters + weight shifts)
- Any Owner contributing > GD 0 increases Political Exposure by +1 (max 3)

### 5.10.2 Quick Mode Mini-Election (R5)

Simplified election for Quick Mode:
- **2 candidates** (not 3) — server picks 2 from `data/elections.json`
- **Contribution:** GD 0 or GD 25,000 only (simplified tier)
- **Bundle duration:** 1 round (instead of 2)
- **Timer:** 15s
- This gives Quick Mode a political moment without the full weight of Standard/Long elections

## 5.11 DEALS Phase

Resolve accepted deals deterministically (see Appendix A2).

## 5.12 EVENTS Phase

Resolve targeted events and rare global events, applying fairness caps.

## 5.13 PRESS Phase

Generate 1-3 Spanish headlines in the match Press Persona style.

## 5.14 AWARDS Phase (Optional)

Server nominates Owners for:
- "Favorito del público"
- "Ampay del día"

Watchers vote (1 tap). Awards applied at end of match.

## 5.15 Special Rounds (Tempo Markers)

Mandatory "moment density" events:

| Mode | Round | Event |
|------|-------|-------|
| Quick | R3 | Guerra de Distrito |
| Quick | R5 | Mini-Eleccion |
| Quick | R6 | Ventana Bancaria |
| Standard | R3 | Guerra de Distrito |
| Standard | R5 | Ventana Bancaria |
| Standard | R9 | Subasta de Prestigio |
| Long | R4 | Guerra de Distrito |
| Long | R8 | Ventana Bancaria |
| Long | R13 | Subasta de Prestigio |

### 5.15.1 Special Round Effects

**Guerra de Distrito:**
- Market Row biased to one district
- Completing a set this round grants +1 Fame

**Mini-Eleccion (Quick R5):**
- 2 candidates, simplified contributions (GD 0 or GD 25,000)
- Bundle lasts 1 round only
- See 5.10.2

**Ventana Bancaria:**
- Loan improves: +GD 200,000 now (instead of +GD 150,000), repay total still Principal + GD 30,000 + (InterestRates × GD 15,000)
- Media/Scandal event weight +10%

**Subasta de Prestigio:**
- Auction win grants +2 Fame
- Watchers see "Portada preview"

---

# PART 6: ECONOMY & SCORING

## 6.1 Resources

| Resource | Range | Description |
|----------|-------|-------------|
| Cash | GD 0+ | Liquid money (Graf Dollar) |
| Debt | GD 0-750,000 | Owed money (cap GD 750,000) |
| Fame | 0+ | Media attention / reputation |
| Compliance | 0-3 | Regulatory standing |
| Political Exposure | 0-3 | Government attention |

## 6.2 Wealth Formula (Primary Win Condition)

```
Wealth = Cash + Σ(PropertyWorth) − Debt

PropertyWorth = BasePrice + (UpgradeLevel × 40,000) + WorthBonuses + PackageWorthBonus
```

**WorthBonuses** include:
- Infrastructure corridor bonus (+GD 100,000 while active)
- Temporary worth bonuses from events
- **PackageWorthBonus**: +GD 75,000 if property is in an active Package (see 7.9)

## 6.3 Rent Calculation

```
Rent = BaseRent
     + (UpgradeLevel × 12,000)
     + DistrictSetBonus
     + MarketSignalBonus
     + TemporaryRentBonus
     + PackageRentBonus
```

### 6.3.1 District Set Bonus
2+ properties in same district: **+GD 5,000 rent per property** in that district.

### 6.3.2 Market Signal Bonus
Each signal modifies rent by tag (±GD 3,000 per property per matching tag, per-property cap ±GD 3,000).

### 6.3.3 Package Rent Bonus
Properties in an active Package: **+GD 6,000 rent per property** in the package (see 7.9).

## 6.4 Market Signals (Tycoon Layer)

Each round reveals 2 signals. Signals defined in `data/signals.json`.

| ID | Name | Tag Effects (units) |
|----|------|---------------------|
| S-01 | Boom Airbnb | turismo +1, costa +1 |
| S-02 | Oficinas flojas | oficinas -1 |
| S-03 | Puerto full | logistica +1 |
| S-04 | Lima de noche | cultura +1 |
| S-05 | Familias compran | residencial +1 |
| S-06 | Burocracia al 200% | gobierno -1 |
| S-07 | Turismo asustado | turismo -1, costa -1 |
| S-08 | Rebote post-crisis | residencial +1, oficinas +1 |
| S-09 | Flete caro | logistica -1 |
| S-10 | Moda Barranco | cultura +1, turismo +1 |

**Rule**: No repeat of same signal in consecutive rounds.
**GD conversion**: Each signal unit = ±GD 3,000 per matching tag per property (server applies).

## 6.5 Operating Costs (Portfolio Overhead)

```
OpsCost = floor((NumProperties + TotalUpgrades) / 4) × 8,000
```

Properties in a Package count as `ceil(count/2)` toward NumProperties for OpsCost (efficiency bonus).

Applied during INCOME phase.

## 6.6 Leader Friction ("Impuesto a la fama")

Applied during INCOME phase:
- Top Wealth quartile: pay **GD 8,000**
- #1 Wealth pays **GD 15,000** if `Wealth > 1.2 × Wealth(#2)`

## 6.7 Forced Sales

If cash < 0 after Income:
1. Sell lowest-basePrice property at **80% BasePrice**
2. Repeat until cash ≥ 0 or no properties remain

---

# PART 7: ACTIONS & DEALS

## 7.1 Develop Action (Major — Wired to Construction Enforcement)

**Base cost**: GD 60,000

**Meter wiring**:
```
DevelopCost = 60,000 + floor((ConstructionEnforcement + 1) / 2) × 20,000
```

| Enforcement Level | Cost (GD) |
|-------------------|-----------|
| 0 | 60,000 |
| 1 | 80,000 |
| 2 | 80,000 |
| 3 | 100,000 |

**Effect**: +GD 12,000 rent permanently, +GD 40,000 to PropertyWorth. Adds +0.05 RiskScore next round.

## 7.2 Deal System (Template-Only, Server-Enforced)

### 7.2.1 Limits
- One outgoing offer per Owner per round
- One counter-offer maximum
- Offers expire at end of DEALS phase

### 7.2.2 Deal Templates (7 Types)

| ID | Template | Description |
|----|----------|-------------|
| 1 | Cash for Property | Buy property for cash |
| 2 | Property Swap | Exchange properties |
| 3 | Bank Loan | Borrow from bank (see 7.2.3) |
| 4 | Election Pact | Support candidate for payment |
| 5 | Non-compete | No bid in district for payment |
| 6 | Security Escort | Protection for payment |
| 7 | Sell Package | Sell entire property package as one transaction (see 7.9) |

### 7.2.3 Bank Loan (Wired to InterestRates)

**Principal**: +GD 150,000 now (+GD 200,000 during Ventana Bancaria)

**Repayment** (scheduled for next INCOME):
```
Repay = Principal + 30,000 + (InterestRates × 15,000)
```
- `InterestRates` captured at loan time
- Record `scheduledRepayAmount` and `scheduledRepayRound`

| InterestRates | Repay (Normal) | Repay (Ventana) |
|---------------|----------------|-----------------|
| 0 | GD 180,000 | GD 230,000 |
| 1 | GD 195,000 | GD 245,000 |
| 2 | GD 210,000 | GD 260,000 |
| 3 | GD 225,000 | GD 275,000 |

**Default**:
- If cannot pay: amount converts to Debt
- Fame -1 with headline "Deudor moroso"

### 7.2.6 Smart Deal Suggestions

Server generates pre-filled deal templates as contextual suggestions:

| Trigger Condition | Suggested Deal |
|-------------------|---------------|
| Another Owner has a property completing your district set | CASH_FOR_PROPERTY |
| Your cash < DevelopCost and you have upgradable properties | BANK_LOAN |
| 2+ Owners target same district repeatedly | NON_COMPETE |
| You own a Package and another Owner wants entry to that category | SELL_PACKAGE |

- Appear as one-tap "Oferta sugerida" cards in Deal Tray
- Player can modify amounts before sending
- Server generates max 2 suggestions per round per Owner

### 7.2.4 Deal Tray UI (v10 Fix)

**CRITICAL**: Never use blocking modals for deals.

**Implementation**:
1. Incoming deals appear in **Deal Tray** (sidebar/toast)
2. Non-blocking: player can continue other actions
3. Tray shows pending deals with Accept/Reject buttons

### 7.2.5 Busy Mode ("Modo Ocupado")

**Toggle**: Player can enable "Modo Ocupado"

**Effect**: Server auto-rejects all incoming deals immediately

**Feedback to sender**: "El jugador está ocupado (Auto-reject)"

**UI Label**: "Modo Concentrado"

## 7.3 Betrayal Mechanic: Romper Pacto

**Eligibility**: Owner has an active Non-compete pact

**Cost**: Once per match

**Effect**:
- Cancel pact immediately
- Fame +1 now
- RiskScore +1.0 next round
- Triggers trophy: `A-BET-01 "Rompiste el Pacto"`

## 7.4 Insurance (Minor Action — Property-Level)

| Upgrade Path | Cost (GD) |
|--------------|-----------|
| None → Basic | 20,000 |
| Basic → Full | 40,000 |
| None → Full | 60,000 |

Mitigation effects defined in Appendix B4.

## 7.5 Security Contract (Minor Action — Owner-Level)

**Cost**: GD 20,000
**Duration**: Until end of next round

**Mitigation**:
- Extortion cash effect reduced by GD 50,000
- Blocks Fame loss from extortion cards

## 7.6 PR / Compliance Actions (Minor)

| Action | Effect |
|--------|--------|
| PR Push | Fame +1 now; RiskScore +0.5 next round |
| Compliance Audit | Compliance +1 (max 3); RiskScore -0.5 next round |

## 7.7 Fame Spend Options (Once Per Round)

| Option | Cost | Effect | Limit |
|--------|------|--------|-------|
| Yape de emergencia | 2 Fame | +GD 60,000 cash | Once per match |
| Control de daños | 2 Fame | Reduce event cash penalty by GD 50,000 | Per event |
| Cortina de humo | 3 Fame | Cancel one Fame loss | Per event |

## 7.8 Error Messages (Spanish)

If action requires cash you don't have:
> **"No te alcanza, causa."**

## 7.9 Property Packaging ("Paquete Inmobiliario")

### 7.9.1 Concept

Group 3+ property cards under a management umbrella to create a consolidated portfolio competitive with expensive individual properties. This is **player-level** packaging (grouping cards for synergy), distinct from the **card-level** portfolio framing (entry cards like "Portafolio 5 Depas" already represent bundled units thematically).

### 7.9.2 Requirements

**Action type:** Major action `PACKAGE` during ACTION phase.

**Conditions:**
- Own 3+ properties sharing a `packageCategory`:
  - `RESIDENTIAL_BUNDLE`: 3+ properties with `residencial` tag
  - `LOGISTICS_HUB`: 3+ properties with `logistica` tag
  - `COMMERCIAL_STRIP`: 3+ properties with `oficinas` or `cultura` tag
- No property already in another package
- Cost: GD 40,000

### 7.9.3 Effects

| Effect | Value |
|--------|-------|
| Rent bonus per property in package | +GD 6,000 |
| Wealth bonus (package worth) | +GD 75,000 |
| OpsCost efficiency | Properties count as `ceil(count/2)` toward NumProperties |
| Unlocks | Deal template #7: SELL_PACKAGE |

### 7.9.4 Dissolution

- Selling or losing any property in the package dissolves it immediately
- Cannot reform a package for **2 rounds** after dissolution

### 7.9.5 Strategic Analysis

**"Many cheap" with packaging:**
3 entry properties: P-01 (200K) + P-03 (250K) + P-04 (280K) = GD 730K + 40K fee = GD 770K total.
Rent: (30K + 35K + 38K) + 18K synergy = GD 121,000/round. OpsCost: 0.

**"One premium" strategy:**
P-13 Edificio Surquillo: GD 750K. Rent: 75,000/round. OpsCost: 0.

Package generates 61% more rent but exposes 3 properties to events (higher RiskScore). Higher reward, higher variance.

### 7.9.6 TypeScript Interface

```typescript
interface PropertyPackage {
  packageId: Id;
  ownerId: Id;
  category: "RESIDENTIAL_BUNDLE" | "LOGISTICS_HUB" | "COMMERCIAL_STRIP";
  propertyIds: Id[];
  createdRound: number;
  rentBonusPerProperty: number;  // 6,000
  worthBonus: number;            // 75,000
  dissolvedRound?: number;       // if dissolved, when
}
```

---

# PART 8: TARGETED EVENT SYSTEM

## 8.1 RiskScore Formula (Explicit)

```
RiskScore =
    1.0
  + 0.35 × NumProperties
  + 0.20 × FameTier
  + 0.20 × WealthTier
  + 0.25 × PoliticalExposure
  - 0.25 × Compliance
  + DistrictRiskBonus
  + AttentionBonus
  - ProtectionBonus
  - RecentTargetShield
  + RumorBonus
```

**Clamp**: [0.5, 3.5]

### 8.1.1 Bonus Breakdown

| Bonus | Condition | Value |
|-------|-----------|-------|
| DistrictRiskBonus | Owns logistica-tag property | +0.2 |
| DistrictRiskBonus | Owns Centro/gobierno-tag | +0.1 |
| DistrictRiskBonus | Many costa-tag properties | +0.1 |
| AttentionBonus | PR Push last round | +0.2 |
| ProtectionBonus | Security Contract active | -0.2 |
| RecentTargetShield | Targeted last round | -0.6 |
| RumorBonus | Watcher rumors (capped) | +0.0 to +0.6 |

### 8.1.2 Tier Calculation

```
FameTier = floor(Fame / 3)  // 0-3
WealthTier = floor(Wealth / 500,000)  // 0-3 typical (GD scale)
```

## 8.2 Event Category Weights

| Category | Base Weight |
|----------|-------------|
| Seguridad | 25% |
| Licencias | 25% |
| Prensa | 25% |
| Incidentes | 20% |
| Global | 5% |

Special rounds can shift weights:
- Ventana Bancaria: Prensa +10%

## 8.3 Fairness Caps (Must Enforce)

| Cap | Rule |
|-----|------|
| FairnessCap | Max GD 150,000 cash damage per Owner per round; overflow becomes rent penalty |
| Expropriation | Max 1 per Owner per match |
| Global shocks | Mode cap; never >1 per 3 rounds |
| Double-target | No same Owner targeted twice in one round |
| Mercy rule | 0 properties → cannot be hit by property-loss events |
| HardHitSpacing | Max 1 event with cash loss ≥ GD 100,000 per 2-round window |
| Scandal bias | ≥60% of Prensa events are Fame-heavy (cash ≤ GD 50,000) |

## 8.4 AutoChoice (Keep Gameplay Fast)

Server auto-resolves event choices:

```
if choice.payCash <= player.cash:
    if choice.payCash <= choice.penaltyValue:
        take "pay" option
    else:
        take "penalty" option
else:
    take "penalty" option
```

**Optional pre-match preference**:
- "Siempre pagar para evitar penalidades: Sí/No"
- If "No": AutoChoice only pays if `payCash < penaltyValue`

## 8.5 Infrastructure Modifiers & Expropriation

### 8.5.1 Infrastructure Modifiers

| ID | Name | Duration |
|----|------|----------|
| INF-01 | Terminal del Metro anunciado | 2 rounds |
| INF-02 | Nueva vía expresa | 2 rounds |

**While INF active on district**:
- `enablesExpropriation = true`
- Rent +GD 5,000 for properties in district
- Worth +GD 100,000 for properties in district
- MapView shows animated corridor ("METRO" / "VIA")

### 8.5.2 E-053 "Anuncio de obras"

**Type**: GLOBAL_TARGET
**Logic**: ACTIVATE_INFRA

**District selection bias**:
- 50% choose among districts in current Market Row
- 50% choose among districts with most owned properties

**Effect**: Activates INF-01 or INF-02 on chosen district for 2 rounds

### 8.5.3 E-051 "Expropiación Municipal"

**Type**: TARGETED
**Condition**: DISTRICT_HAS_INFRA

**Resolution rule (no "no-op")**:
1. If ≥1 district has `enablesExpropriation=true`:
   - Target Owner who owns property in enabled district
   - One reroll of target owner if needed
2. If no enabled districts:
   - Resolve fallback **E-051B "Aviso de derecho de vía"**

### 8.5.4 E-051B "Aviso de derecho de vía" (Fallback)

**Effects**:
- Cash -GD 50,000
- `tradeLockedUntilRound = currentRound + 2` on random owned property

**Headline theme**: Blueprint/"trazo"

---

# PART 9: ELECTIONS & WORLD METERS

## 9.1 World Meters

| Meter | Range | Effects |
|-------|-------|---------|
| Political Stability | 0-3 | Affects global shock probability |
| BCRP Independence | 0-3 | Affects inflation events |
| Security | 0-3 | Affects extortion event weight |
| Inflation | 0-3 | Affects cash value |
| Interest Rates | 0-3 | Affects Bank Loan repayment |
| Construction Enforcement | 0-3 | Affects Develop cost |

## 9.2 Election Candidates

Defined in `data/elections.json`. Bundles apply for 2 rounds after election (1 round for Quick Mode Mini-Elections).

**Contribution tiers:** GD 0 / GD 25,000 / GD 50,000 (Quick Mode: GD 0 / GD 25,000 only).

### 9.2.1 Candidate Examples

**C-01 "General Orden"** — "Mano dura, pero con licencia"
- Meters: Security +1, ConstructionEnforcement +1
- Weights: Extortion -10%, Licencias +10%

**C-02 "Tecnocrata Pro"** — "Estabilidad, por favor"
- Meters: BCRP +1, InterestRates -1, PoliticalStability +1
- Weights: (none)

**C-03 "Shock Reformista"** — "Corta la grasa"
- Meters: PoliticalStability +1, InterestRates +1, Inflation -1
- Weights: Prensa +10%

**C-04 "Estado Expansivo"** — "Más Estado, más obra"
- Meters: Inflation +1, BCRP -1, PoliticalStability -1
- Weights: Licencias +15%

## 9.3 Political Exposure Effects

Each point of Political Exposure:
- +0.25 to RiskScore
- Increased targeting by government-related events

---

# PART 10: PRESS SYSTEM

## 10.1 Press Personas (5)

| Persona | Style | Tone |
|---------|-------|------|
| Diario serio | Formal newspaper | Neutral, factual |
| Portada chicha | Tabloid | Sensational, gossipy |
| Programa de la tarde | TV talk show | Dramatic, opinionated |
| Radio con llamadas | Call-in radio | Populist, reactive |
| Boletín municipal | Official bulletin | Bureaucratic, dry |

## 10.2 Headline Generation

Headlines are template-driven Spanish strings.

**Template example**:
```
"{OWNER} compra {PROPERTY} por ${AMOUNT} — ¿Qué sabe que nosotros no?"
```

**Constraints**:
- Headlines ≤70 characters
- Card bodies ≤120 characters

## 10.3 Spanish Humor Guidelines

**Allowed colloquialisms**:
causa, al toque, asu, palta, con fe, chambea, yape, chamba, jato, chela

**NOT allowed**:
- Slurs
- Harassment
- Targeting protected characteristics
- Political statements about real current events

## 10.4 Real-World Inspiration Rules

Use **partial-name flavor tokens** only.

**Example**: "estilo Brescia" not "Mario Brescia dijo..."

Never frame as new factual claims about real people.

---

# PART 11: TECHNOLOGY ARCHITECTURE

## 11.1 Open Source Foundation Matrix

See **Appendix O** for complete evaluation with repository links.

### 11.1.1 Primary Recommendations

| Component | Solution | Repository | License |
|-----------|----------|------------|---------|
| Server Framework | Colyseus | [colyseus/colyseus](https://github.com/colyseus/colyseus) | MIT |
| Client Engine | Godot 4.x | [godotengine/godot](https://github.com/godotengine/godot) | MIT |
| TypeScript in Godot | GodotJS | [aspect-build/aspect-cli](https://github.com/aspect-build/aspect-cli) | MIT |
| Card UI Components | godot-card-game-framework | [db0/godot-card-game-framework](https://github.com/db0/godot-card-game-framework) | AGPL3 |

### 11.1.2 Why Colyseus

- Built-in authoritative state synchronization
- Room-based architecture matches game design
- TypeScript-native (matches server language)
- Battle-tested WebSocket handling
- Schema-based state with delta compression

## 11.2 Client Architecture (Godot 4.x)

### 11.2.1 Renderer
- **Compatibility** renderer for widest device support (WebGL2)
- Avoid threads
- Avoid heavy shaders
- 2D UI-first approach

### 11.2.2 UI Split
- **Admin/menus**: Graf Design System tokens + Material Symbols
- **Match**: "Imperio" aesthetics (newspaper, skyline, district overlays)

### 11.2.3 Input Handling
- Mobile text inputs (Profile/Chat) use **HTML DOM Overlays**
- Avoid Godot/WebGL virtual keyboard issues

## 11.3 Server Architecture (Node.js + Colyseus)

### 11.3.1 Core Stack
```
Node.js (v20+)
├── Colyseus (WebSocket server)
├── TypeScript (strict mode)
├── PostgreSQL (persistence)
└── Redis (optional: rate limiting, sessions)
```

### 11.3.2 Room Types
```typescript
// Colyseus room definition
export class ImperioRoom extends Room<ImperioState> {
  maxClients = 20;

  onCreate(options: RoomOptions) {
    this.setState(new ImperioState());
    this.setMetadata({ mode: options.mode });
  }
}
```

## 11.4 Database (PostgreSQL)

### 11.4.1 Core Tables
- `accounts` - User accounts
- `matches` - Match history
- `achievements` - Unlocked achievements
- `dynasties` - Team/dynasty stats

### 11.4.2 State Persistence
- Match snapshots stored as JSONB
- Replay capability for disputes

## 11.5 Infrastructure

### 11.5.1 MVP Setup
```
VPS (4GB RAM minimum)
├── Nginx (TLS proxy)
├── Node.js (Colyseus server)
├── PostgreSQL
└── Local file storage (share assets)
```

### 11.5.2 Scale-Up Path
- Redis for session sharing
- S3 for share assets
- Multiple Colyseus processes with matchmaker

---

# PART 12: SECURITY & ANTI-ABUSE

## 12.1 Phase Gating

Server enforces strict phase transitions:
- Actions only accepted during correct phase
- Invalid phase actions rejected with error

## 12.2 Validation Rules

| Rule | Implementation |
|------|----------------|
| Bid validation | Amount ≤ player cash |
| Action validation | Player has required resources |
| Deal validation | Both parties have assets |
| Property ownership | Verified before any transfer |

## 12.3 JIT Round Seed (v10 Security Fix)

**Problem**: Pre-shared seeds allow RNG prediction

**Solution**:
```typescript
// Server-side only
const serverSecret = crypto.randomBytes(32).toString('hex');

function generateRoundSeed(matchSeed: string, roundNumber: number): string {
  return sha256(`${matchSeed}|${serverSecret}|${roundNumber}`);
}
```

**Rule**: `serverSecret` NEVER sent to client

## 12.4 Rate Limiting

| Action | Limit |
|--------|-------|
| Bid submission | 1 per round |
| Deal offer | 1 per round |
| Mood vote | 1 per round |
| Rumor | 1 per match |
| Reconnect attempts | 5 per minute |

## 12.5 Idempotency

- All client messages include `idempotencyKey` (UUID)
- Server rejects duplicate keys
- Monotonic nonce per session

## 12.6 Reconnection Handling

| Scenario | Behavior |
|----------|----------|
| Disconnect < 60s | Full reconnect, resume state |
| Disconnect 60s-5min | Autopilot continues (Bot heuristics) |
| Disconnect > 5min | Seat may be locked (host rules) |

---

# PART 13: GRAF DOLLAR METAGAME ECONOMY

## 13.1 Currency Model — Prestige Leaderboard

**Core principle:** Graf Dollar (GD) is a **prestige currency for lifetime wealth ranking**. It does NOT grant in-match advantage. Matches are always fair — everyone starts equal.

| Layer | How It Works |
|-------|-------------|
| **In-match** | All players start with identical GD (500K/650K/800K by mode). Match plays normally. At match end, final Wealth is calculated. |
| **Persistent Wallet** | Match outcomes ADD GD to your lifetime balance. Tracked in PostgreSQL double-entry ledger per `graf_dollar.txt`. |
| **Leaderboard** | Ranked by accumulated lifetime GD. Players display any custom name they choose. Dynasty name shown alongside. |

**What GD is for:**
- Lifetime wealth leaderboard ranking (primary prestige driver)
- Display name + dynasty branding on the public ladder
- Optional cosmetic unlocks (badges, crests, titles)
- Player-to-player transfers/loans (Phase 2/3 per graf_dollar.txt)

**What GD is NOT for:**
- Buying in-match advantages
- Starting with more cash than other players
- Unlocking stronger properties or actions
- Any pay-to-win mechanic

## 13.2 Match-to-Wallet Flow

At match end, each player's final in-match Wealth is converted to persistent GD and credited to their wallet via the ledger system.

| Outcome | Persistent GD Earned |
|---------|---------------------|
| Final Wealth (always) | Your end-of-match Wealth added to wallet |
| 1st place bonus | +GD 200,000 |
| 2nd place bonus | +GD 100,000 |
| 3rd place bonus | +GD 50,000 |
| Achievement unlock | +GD 20,000-100,000 depending on rarity |

**Design rationale:** Awarding actual final Wealth (not just flat bonuses) means every in-match decision matters for the leaderboard. A 1st-place finish with GD 3M Wealth earns massively more than a 1st-place with GD 1M. This incentivizes aggressive, high-Wealth play.

**Ledger Architecture:** Uses `ledger_tx` + `ledger_entry` double-entry system from `graf_dollar.txt`. Match rewards are `tx_type='award'` with `idempotency_key='award:{match_id}:{user_id}:wealth'`. System account mints all GD.

**New player welcome:** GD 0 starting balance. No GD needed to play.

## 13.3 Leaderboard System

### 13.3.1 Tiered Leagues

| League | GD Threshold | Badge Color |
|--------|-------------|-------------|
| Inquilino (Tenant) | 0 - 999,999 | Gray |
| Propietario (Owner) | 1,000,000 - 9,999,999 | Bronze |
| Inversionista (Investor) | 10,000,000 - 49,999,999 | Silver |
| Magnate | 50,000,000 - 199,999,999 | Gold |
| Emperador (Emperor) | 200,000,000+ | Diamond |

**Threshold rationale:** Average Quick Mode final Wealth ~ GD 800K-1.5M. Winner ~ GD 2-3M. Propietario requires ~1-5 matches. Inversionista ~10-30. Magnate ~50-100. Emperador ~150+ matches of strong play.

### 13.3.2 Seasonal Decay

- Monthly soft reset: all balances above GD 5,000,000 decay by 10%
- Forces active play to maintain ranking
- Season rewards: top 100 get exclusive cosmetic badge

### 13.3.3 Contextual Views

| View | Description |
|------|-------------|
| Friends | Compare against your circle |
| Regional | Lima districts — ties into game theme |
| Dynasty | Aggregate GD of all players in same dynasty |
| All-time | Permanent hall of fame |

### 13.3.4 Social Display

GD balance and league badge shown on:
- Profile card
- Portada share cards (end-of-match)
- Lobby player list
- Quick Chat name tag

**Virality angle:** The Dynasty leaderboard is the strongest viral hook — players recruit friends into their dynasty to climb the collective rankings. This aligns with the referral system (Section 3.5.4).

## 13.4 Cosmetic Spending (Never Gameplay Advantage)

| Phase | Available Spending |
|-------|-------------------|
| Phase 1 (MVP) | Profile badges, team crests, custom titles |
| Phase 2 | Player-to-player GD transfers (gifting, alliance signaling) |
| Phase 3 | Player-to-player GD loans (social contracts) |

## 13.5 Database Integration

### 13.5.1 New/Updated Tables

| Table | Source | Purpose |
|-------|--------|---------|
| `ledger_tx` | graf_dollar.txt §4.1.1 | Transaction header (award, transfer, purchase, etc.) |
| `ledger_entry` | graf_dollar.txt §4.1.2 | Double-entry lines (sum = 0 per tx) |
| `balance_cache` | graf_dollar.txt §4.1.3 | Derived balance for performance |
| `loan_contract` | graf_dollar.txt §4.1.4 | Phase 3 player loans |

### 13.5.2 Idempotency Keys

| Operation | Key Format |
|-----------|-----------|
| Match wealth award | `award:{match_id}:{user_id}:wealth` |
| Placement bonus | `award:{match_id}:{user_id}:placement:{rank}` |
| Achievement bonus | `award:{match_id}:{user_id}:achievement:{achievement_id}` |
| Transfer | `xfer:{sender_id}:{client_request_id}` |
| Cosmetic purchase | `buy:{user_id}:{order_id}` |

---

# APPENDICES

---

# APPENDIX A: DATA SCHEMAS & LOGIC

## A1: TypeScript Types (Authoritative)

```typescript
// === Core Types ===
type Id = string;

type RoomType = "PUBLIC" | "FRIENDS";
type Role = "OWNER" | "WATCHER";
type Phase =
  | "REVEAL"
  | "BID"
  | "INCOME_CALC"
  | "VISUAL_LEDGER"
  | "ACTION"
  | "MOOD"
  | "VOTE"
  | "DEALS"
  | "EVENTS"
  | "PRESS"
  | "AWARDS"
  | "SNAPSHOT"
  | "END";

type InsuranceLevel = "NONE" | "BASIC" | "FULL";
type BuildQuality = "STANDARD" | "PREMIUM";

// === Owner State ===
interface OwnerState {
  ownerId: Id;
  handle: string;
  displayName: string;
  realName?: string;
  realNameVisibility: "HIDDEN" | "VISIBLE";
  team: string;
  teamSet: "REAL_LABEL" | "PARODY";
  color: string;

  cash: number;
  debt: number;
  fame: number;
  compliance: 0 | 1 | 2 | 3;
  politicalExposure: 0 | 1 | 2 | 3;

  protectedUntilRound?: number;
  brokePactUsed: boolean;
  yapeUsed: boolean;
  lastTargetedRound?: number;

  preferAutoPay: boolean;

  properties: Id[];
  achievements: string[];
}

// === Property State ===
interface PropertyState {
  propertyId: Id;
  nombre_es: string;
  district: string;
  tags: string[];
  basePrice: number;       // GD (e.g. 200000)
  baseRent: number;        // GD (e.g. 30000)
  upgradeSlotsMax: 0 | 1 | 2;
  upgradeLevel: 0 | 1 | 2;
  insurance: InsuranceLevel;
  buildQuality: BuildQuality;
  ownerId?: Id;
  packageId?: Id;          // v12: if part of a PropertyPackage
  packageCategory?: PackageCategory;  // v12: eligible package type
  tempRentBonus?: Array<{ delta: number; untilRound: number }>;
  tempWorthBonus?: Array<{ delta: number; untilRound: number }>;
  tradeLockedUntilRound?: number;
}

// === Property Package (v12) ===
type PackageCategory = "RESIDENTIAL_BUNDLE" | "LOGISTICS_HUB" | "COMMERCIAL_STRIP";

interface PropertyPackage {
  packageId: Id;
  ownerId: Id;
  category: PackageCategory;
  propertyIds: Id[];
  createdRound: number;
  rentBonusPerProperty: number;  // 6,000
  worthBonus: number;            // 75,000
  dissolvedRound?: number;
}

// === Mood Option (v12) ===
type MoodEffectType =
  | "RENT_ALL"
  | "DEVELOP_COST"
  | "RISK_ALL"
  | "SUPPRESS_EVENT"
  | "FAME_MULTIPLIER"
  | "COMPLIANCE_BONUS"
  | "RENT_TIER";

interface MoodOption {
  id: Id;
  nombre_es: string;
  effect: {
    type: MoodEffectType;
    delta?: number;
    multiplier?: number;
    threshold?: number;
    duration: number;
  };
  sentiment: "POSITIVE" | "NEGATIVE" | "NEUTRAL";
}

// === Quick Chat (v12) ===
interface QuickChatMessage {
  phraseId: string;       // "QC-01"
  fromOwnerId: Id;
  districtParam?: string; // for QC-11
  timestamp: number;
}

// === Deal State ===
type DealTemplateId =
  | "CASH_FOR_PROPERTY"
  | "SWAP"
  | "BANK_LOAN"
  | "ELECTION_PACT"
  | "NON_COMPETE"
  | "SECURITY_ESCORT"
  | "SELL_PACKAGE";  // v12: sell entire property package

type DealStatus =
  | "PENDING"
  | "COUNTERED"
  | "ACCEPTED"
  | "REJECTED"
  | "EXPIRED"
  | "RESOLVED"
  | "INVALID";

interface DealState {
  dealId: Id;
  templateId: DealTemplateId;
  fromOwnerId: Id;
  toOwnerId: Id;
  createdRound: number;
  expiresAtPhase: "DEALS";
  status: DealStatus;
  fields: Record<string, any>;
  counterFields?: Record<string, any>;

  // BANK_LOAN only
  scheduledRepayAmount?: number;
  scheduledRepayRound?: number;
}

// === Event Result ===
type FameSpendUsed = "YAPE" | "CONTROL_DANOS" | "CORTINA";

interface EventResult {
  cardId: Id;
  targetOwnerId?: Id;
  targetPropertyId?: Id;
  effectsApplied: {
    cashDelta: number;
    debtDelta: number;
    fameDelta: number;
    rentPenaltyApplied?: Array<{
      propertyId: Id;
      delta: number;
      duration: number
    }>;
    meterDelta?: Record<string, number>;
  };
  mitigationsApplied: {
    insuranceUsed?: InsuranceLevel;
    securityContractUsed?: boolean;
    compliancePrevented?: boolean;
    fameSpendUsed?: FameSpendUsed;
    capsApplied?: string[];
    autoChoiceTaken?: string;
  };
  headlines: string[];
}

// === Match State Snapshot ===
interface MatchStateSnapshot {
  roomId: Id;
  mode: "QUICK" | "STANDARD" | "LONG";
  round: number;
  phase: Phase;
  phaseEndsAtTs: number;

  owners: OwnerState[];
  properties: PropertyState[];
  packages: PropertyPackage[];  // v12
  deals: DealState[];

  marketRow: Id[];
  marketSignals: Array<{
    id: Id;
    nombre_es: string;
    tagMods: Record<string, number>;
  }>;
  moodOptions?: MoodOption[];  // v12: typed
  activeMoodEffect?: MoodOption;  // v12: currently active mood
  electionOptions?: Array<{
    id: Id;
    nombre_es: string;
    bundle: any;
  }>;

  meters: Record<string, number>;

  infra?: Array<{
    infraId: string;
    district: string;
    untilRound: number;
    rentDelta: number;     // GD 5,000
    worthDelta: number;    // GD 100,000
    enablesExpropriation: boolean;
  }>;

  pressPersonaId: Id;
  pressTicker: string[];
  lastEventResults?: EventResult[];

  watcherCount: number;
}

// === Tag Enums (Closed Set) ===
type EconomicTag =
  | "turismo"
  | "oficinas"
  | "logistica"
  | "residencial"
  | "gobierno"
  | "cultura"
  | "costa";

type FlavorTag =
  | "premium_area"
  | "corredor_obra"
  | "zona_caliente";

type PropertyTag = EconomicTag | FlavorTag;
```

## A2: Deal Engine Rules (Deterministic)

### Lifecycle
1. Each Owner can have at most 1 outgoing offer per round
2. Offer expires at end of DEALS phase of same round
3. One counter-offer maximum

### Resolution Order (DEALS Phase)
1. Expire stale offers
2. Resolve ACCEPTED deals deterministically (sorted by `dealId`)
3. If referenced property not owned by sender → deal becomes INVALID

### Conflict Rules
- A property may be transferred at most once per round via deals
- If two accepted deals transfer same property:
  - Lowest `dealId` resolves
  - Others become INVALID

### Forced Sales
- Occur during INCOME
- Can invalidate later deals

## A3: State Machine (Fully Enumerated)

```
LOBBY
  ↓
MATCH_INIT
  ↓
┌─────────────────────────────────────────┐
│ FOR EACH ROUND:                         │
│   REVEAL                                │
│     ↓                                   │
│   BID                                   │
│     ↓                                   │
│   INCOME_CALC                           │
│     ↓                                   │
│   VISUAL_LEDGER (4.0s blocking)         │
│     ↓                                   │
│   ACTION                                │
│     ↓                                   │
│   MOOD (or VOTE on election rounds)     │
│     ↓                                   │
│   DEALS                                 │
│     ↓                                   │
│   EVENTS                                │
│     ↓                                   │
│   PRESS                                 │
│     ↓                                   │
│   [AWARDS] (optional)                   │
│     ↓                                   │
│   SNAPSHOT                              │
└─────────────────────────────────────────┘
  ↓
END
```

## A4: Market Signals List

See `data/signals.json` for authoritative source.

| ID | Name (Spanish) | Tag Effects |
|----|----------------|-------------|
| S-01 | Boom Airbnb | turismo +1, costa +1 |
| S-02 | Oficinas flojas | oficinas -1 |
| S-03 | Puerto full | logistica +1 |
| S-04 | Lima de noche | cultura +1 |
| S-05 | Familias compran | residencial +1 |
| S-06 | Burocracia al 200% | gobierno -1 |
| S-07 | Turismo asustado | turismo -1, costa -1 |
| S-08 | Rebote post-crisis | residencial +1, oficinas +1 |
| S-09 | Flete caro | logistica -1 |
| S-10 | Moda Barranco | cultura +1, turismo +1 |

## A5: Elections Data

See `data/elections.json` for authoritative source.

### Candidate Schema
```typescript
interface Candidate {
  id: string;           // "C-01"
  name: string;         // "General Orden"
  slogan: string;       // "Mano dura, pero con licencia"
  meters: Record<string, number>;  // { "Security": 1 }
  weightMods: Record<string, number>;  // { "Extortion": -10 }
}
```

### Bundle Duration
All election effects last **2 rounds** after winner determined.

## A6: Achievements List

| ID | Name (Spanish) | Trigger |
|----|----------------|---------|
| A-TUT-01 | Primer Casero | Complete tutorial |
| A-BET-01 | Rompiste el Pacto | Use Romper Pacto |
| A-WIN-01 | Magnate | Win a match |
| A-WIN-05 | Rey de Lima | Win 5 matches |
| A-FAME-01 | Famoso | Reach 5 Fame |
| A-FAME-10 | Leyenda | Reach 10 Fame |
| A-PROP-05 | Portafolio | Own 5 properties |
| A-PROP-10 | Imperio | Own 10 properties |
| A-SET-01 | Dueño del Barrio | Complete a district set |
| A-DEAL-01 | Negociante | Complete 5 deals |
| A-DEAL-10 | Tiburón | Complete 10 deals |
| A-SURV-01 | Sobreviviente | Survive with 0 cash |
| A-EVENT-01 | Víctima | Get hit by 5 events |
| A-INS-01 | Precavido | Fully insure a property |

## A7: Property Deck

See `data/properties.json` for authoritative source.

### Deck Rules
- Finite list
- Market Row draws without replacement
- When deck ends: reshuffle discard with deterministic shuffle using `matchSeed`

### Property Schema
```typescript
interface Property {
  id: string;           // "P-01"
  nombre_es: string;    // "Portafolio Depas Los Olivos"
  district: string;     // "Independencia/Los Olivos"
  tags: string[];       // ["residencial"]
  basePrice: number;    // 200000 (GD)
  baseRent: number;     // 30000 (GD)
  upgradeSlotsMax: 0 | 1 | 2;
  buildQuality: "STANDARD" | "PREMIUM";
  packageCategory?: "RESIDENTIAL_BUNDLE" | "LOGISTICS_HUB" | "COMMERCIAL_STRIP";
}
```

**Property card concept:** Each card represents a **real estate operation**, not necessarily a single unit. Lower-end cards represent bundled portfolios of smaller assets (e.g., "Portafolio 5 Depas Los Olivos" = five apartments packaged as one investment card). Higher-end cards represent individual trophy assets (e.g., "Torre Financiera" = one floor of a premium office tower). All cards work identically in-game.

### Expanded Property Deck (25 Properties)

**Tier structure:** Entry (200K-400K, ~13-15% yield, 7 properties), Mid (450K-750K, ~10-12% yield, 6 properties), Upper (800K-1.3M, ~8-10% yield, 7 properties), Premium (1.5M-2M, ~6.5-7.5% yield, 5 properties).

| ID | Name | District | Tags | Price (GD) | Rent (GD) | Slots |
|----|------|----------|------|------------|-----------|-------|
| P-01 | Portafolio Depas Los Olivos | Independencia/Los Olivos | residencial | 200,000 | 30,000 | 1 |
| P-02 | Mini-Locales San Juan | Ate/SJL | logistica, residencial | 220,000 | 32,000 | 1 |
| P-03 | Lotes Puente Piedra | Independencia/Los Olivos | residencial | 250,000 | 35,000 | 1 |
| P-04 | Depas Costa Chorrillos | Chorrillos | residencial, costa | 280,000 | 38,000 | 1 |
| P-05 | Galeria Comercial Centro | Centro | cultura, gobierno | 300,000 | 40,000 | 1 |
| P-06 | Depas Pueblo Libre | Pueblo Libre | residencial, cultura | 350,000 | 45,000 | 1 |
| P-07 | Taller Automotriz Chorrillos | Chorrillos | logistica, costa | 400,000 | 50,000 | 1 |
| P-08 | Strip Mall Lince | Jesus Maria/Lince | residencial, gobierno | 450,000 | 55,000 | 1 |
| P-09 | Bodega Industrial Ate | Ate/SJL | logistica | 500,000 | 60,000 | 1 |
| P-10 | Portafolio Depas San Miguel | Magdalena/San Miguel | residencial, costa | 550,000 | 65,000 | 1 |
| P-11 | Casona Restaurada Barranco | Barranco | cultura, turismo | 600,000 | 66,000 | 1 |
| P-12 | Almacen Callao Puerto | Callao | logistica, zona_caliente | 700,000 | 73,000 | 2 |
| P-13 | Edificio Surquillo | Surquillo | residencial | 750,000 | 75,000 | 2 |
| P-14 | Casa Residencial Surco | Surco | residencial | 800,000 | 76,000 | 2 |
| P-15 | Loft Artistico Barranco | Barranco | cultura, turismo, costa | 850,000 | 80,000 | 2 |
| P-16 | Residencial La Molina | La Molina | residencial, premium_area | 900,000 | 82,000 | 2 |
| P-17 | Depa Vista Mar Miraflores | Miraflores | residencial, costa, premium_area | 1,000,000 | 90,000 | 2 |
| P-18 | Oficina Miraflores | Miraflores | oficinas, costa | 1,100,000 | 95,000 | 2 |
| P-19 | Centro Logistico Callao | Callao | logistica, zona_caliente | 1,200,000 | 100,000 | 2 |
| P-20 | Edificio Residencial Surco | Surco | residencial | 1,300,000 | 104,000 | 2 |
| P-21 | Oficina San Isidro Prime | San Isidro | oficinas, gobierno, premium_area | 1,500,000 | 112,000 | 2 |
| P-22 | Penthouse Barranco | Barranco | residencial, cultura, costa | 1,600,000 | 115,000 | 2 |
| P-23 | Casa La Molina Grande | La Molina | residencial, premium_area | 1,700,000 | 119,000 | 2 |
| P-24 | Penthouse San Isidro | San Isidro | residencial, premium_area | 1,800,000 | 122,000 | 2 |
| P-25 | Torre Financiera San Isidro | San Isidro | oficinas, gobierno, premium_area | 2,000,000 | 130,000 | 2 |

**Tag distribution:** residencial (15), logistica (5), cultura (5), oficinas (3), costa (7), turismo (3), gobierno (4), premium_area (6), zona_caliente (2).

**District set bonus opportunities (2+ in same district):** Independencia/Los Olivos (2), Ate/SJL (2), Chorrillos (2), Barranco (3), Callao (2), Surco (2), La Molina (2), Miraflores (2), San Isidro (3).

**Deck sizing:** 25 properties / 5 per Market Row = 5 rounds without reshuffling (covers Quick Mode).

## A8: Bot Heuristics (NEW in v10)

**Role**: Fill empty seats in Tutorial/Public matches; handle disconnects.

### H1: Economy Safety
```typescript
function shouldPass(bot: OwnerState): boolean {
  return bot.cash < 100_000;  // GD 100K reserve
}

function maxBid(bot: OwnerState): number {
  return Math.max(0, bot.cash - 100_000);  // Keep GD 100K reserve
}
```

### H2: Bidding Logic
```typescript
function calculateBid(bot: OwnerState, property: PropertyState): number {
  const valuation = property.basePrice + (property.baseRent * 2);
  const maxAffordable = bot.cash - 50_000;  // Keep GD 50K minimum
  return Math.min(valuation, maxAffordable);
}

// Tutorial Override
function tutorialBid(round: number): number {
  if (round === 1) return 50_000;  // Always lose to human (bid below cheapest property)
  return calculateBid(this, property);
}
```

### H3: Deal Logic
- **Incoming**: Always REJECT (reduces complexity)
- **Outgoing**: Never initiate

### H4: Event Choices
```typescript
function handleEventChoice(cost: number, penalty: number): string {
  if (this.cash >= cost) return "PAY";
  return "PENALTY";
}
```

### H5: Action Selection
```typescript
function selectAction(bot: OwnerState): { major: string; minor?: string } {
  if (bot.cash < 100_000) return { major: "PASS" };

  // Major: Develop if affordable
  const developCost = getDevelopCost();  // GD 60K-100K
  let major = "PASS";
  if (bot.cash >= developCost && bot.properties.length > 0) {
    major = "DEVELOP";
  }

  // Minor: Insure if we have uninsured properties and enough cash
  let minor: string | undefined;
  const hasUninsured = bot.properties.some(p => getProperty(p).insurance === "NONE");
  if (hasUninsured && bot.cash >= developCost + 20_000) {
    minor = "INSURE";
  }

  return { major, minor };
}
```

## A9: Open Source Integration Guide (NEW in v11)

### Colyseus Server Setup

```typescript
// server/src/rooms/ImperioRoom.ts
import { Room, Client } from "colyseus";
import { ImperioState } from "./schema/ImperioState";

export class ImperioRoom extends Room<ImperioState> {
  maxClients = 20;

  onCreate(options: { mode: string }) {
    this.setState(new ImperioState());

    // Set up phase transitions
    this.clock.start();

    // Message handlers
    this.onMessage("bid", (client, data) => {
      this.handleBid(client, data);
    });

    this.onMessage("action", (client, data) => {
      this.handleAction(client, data);
    });
  }

  onJoin(client: Client, options: any) {
    const owner = this.state.addOwner(client.sessionId, options);
    client.send("joined", { ownerId: owner.ownerId });
  }

  onLeave(client: Client, consented: boolean) {
    if (!consented) {
      // Allow reconnection
      this.allowReconnection(client, 60);
    } else {
      this.state.removeOwner(client.sessionId);
    }
  }
}
```

### Colyseus State Schema

```typescript
// server/src/rooms/schema/ImperioState.ts
import { Schema, type, MapSchema, ArraySchema } from "@colyseus/schema";

export class OwnerSchema extends Schema {
  @type("string") ownerId: string;
  @type("string") handle: string;
  @type("number") cash: number = 500_000;  // GD — set per mode (500K/650K/800K)
  @type("number") debt: number = 0;
  @type("number") fame: number = 0;
  @type("number") compliance: number = 1;
  @type(["string"]) properties = new ArraySchema<string>();
  @type(["string"]) packages = new ArraySchema<string>();  // v12: package IDs
}

export class PropertySchema extends Schema {
  @type("string") propertyId: string;
  @type("string") nombre_es: string;
  @type("string") district: string;
  @type("number") basePrice: number;
  @type("number") baseRent: number;
  @type("number") upgradeLevel: number = 0;
  @type("string") ownerId: string = "";
  @type("string") insurance: string = "NONE";
}

export class ImperioState extends Schema {
  @type("string") phase: string = "LOBBY";
  @type("number") round: number = 0;
  @type({ map: OwnerSchema }) owners = new MapSchema<OwnerSchema>();
  @type({ map: PropertySchema }) properties = new MapSchema<PropertySchema>();
  @type(["string"]) marketRow = new ArraySchema<string>();
}
```

### Godot Client Connection

```gdscript
# client/scripts/network/ColyseusClient.gd
extends Node

var client: WebSocketClient
var room_state: Dictionary = {}

signal connected
signal state_changed(state)
signal phase_changed(phase)

func _ready():
    client = WebSocketClient.new()
    client.connect("connection_established", self, "_on_connected")
    client.connect("data_received", self, "_on_data")

func join_room(room_id: String, options: Dictionary):
    var url = "ws://localhost:2567/" + room_id
    client.connect_to_url(url)

func _on_connected(protocol: String):
    emit_signal("connected")

func _on_data():
    var data = client.get_peer(1).get_packet().get_string_from_utf8()
    var message = JSON.parse(data).result

    match message.type:
        "STATE_SNAPSHOT":
            room_state = message.payload
            emit_signal("state_changed", room_state)
        "PHASE_CHANGE":
            emit_signal("phase_changed", message.payload.phase)
        "OP_DELTA":
            _apply_delta(message.changes)

func _apply_delta(changes: Array):
    for change in changes:
        match change.op:
            "SET_CASH":
                room_state.owners[change.id].cash = change.val
            "SET_OWNER":
                room_state.properties[change.id].ownerId = change.val
    emit_signal("state_changed", room_state)

func send_bid(property_id: String, amount: int):
    _send({
        "type": "bid",
        "propertyId": property_id,
        "amount": amount
    })

func send_action(action_type: String, params: Dictionary = {}):
    _send({
        "type": "action",
        "actionType": action_type,
        "params": params
    })

func _send(data: Dictionary):
    var json = JSON.print(data)
    client.get_peer(1).put_packet(json.to_utf8())
```

---

# APPENDIX B: EVENT SYSTEM

## B0: Event Deck Patches

### Added in v9
- **E-053 "Anuncio de obras"**: Activates INF corridor

### Modified in v9
- **E-051 "Expropiación Municipal"**: Now has fallback to E-051B

## B1: Targeted Event Deck (Complete YAML)

```yaml
# ============================================
# IMPERIO INMOBILIARIO - TARGETED EVENT DECK
# 52+ Events with Spanish Headlines
# ============================================

# -----------------------------
# CATEGORY: SEGURIDAD (25%)
# -----------------------------
events:
  - id: E-001
    title_es: "Extorsión telefónica"
    category: SEGURIDAD
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -100000
    mitigation:
      SECURITY_CONTRACT: { cash: -50000 }
    headlines:
      - "Dueño de {PROPERTY} recibe llamadas sospechosas"
      - "¿Cupos en {DISTRICT}? Vecinos preocupados"

  - id: E-002
    title_es: "Robo en propiedad"
    category: SEGURIDAD
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -100000
      rentDelta: -10000
      rentDuration: 1
    mitigation:
      SECURITY_CONTRACT: { cash: -50000, rentDelta: 0 }
    headlines:
      - "Delincuentes atacan {PROPERTY}"
      - "Ola de robos en {DISTRICT}"

  - id: E-003
    title_es: "Vandalismo"
    category: SEGURIDAD
    type: TARGETED
    severity: LOW
    effects:
      cash: -50000
    headlines:
      - "Grafitis aparecen en fachada de {PROPERTY}"

  - id: E-004
    title_es: "Invasión de terreno"
    category: SEGURIDAD
    type: TARGETED
    severity: HIGH
    condition: "property.tags.includes('logistica')"
    effects:
      cash: -150000
      rentDelta: -20000
      rentDuration: 2
    mitigation:
      SECURITY_CONTRACT: { cash: -100000 }
    headlines:
      - "Invasores toman parte de {PROPERTY}"
      - "Crisis en {DISTRICT}: familias ocupan almacén"

  - id: E-005
    title_es: "Amenaza de bomba"
    category: SEGURIDAD
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -50000
      rentDelta: -10000
      rentDuration: 1
    headlines:
      - "Evacuación en {PROPERTY} por amenaza"

  - id: E-006
    title_es: "Cupo sindical"
    category: SEGURIDAD
    type: TARGETED
    severity: MEDIUM
    condition: "property.upgradeLevel > 0"
    effects:
      cash: -100000
    mitigation:
      SECURITY_CONTRACT: { cash: -50000, fameDelta: 0 }
    headlines:
      - "Sindicato exige pago a {OWNER}"
      - "Obras en {PROPERTY} paralizadas por cupo"

  - id: E-007
    title_es: "Protección forzada"
    category: SEGURIDAD
    type: TARGETED
    severity: HIGH
    effects:
      cash: -150000
      fame: -1
    mitigation:
      SECURITY_CONTRACT: { cash: -50000, fame: 0 }
    headlines:
      - "{OWNER} cede ante presión"
      - "Seguridad privada impuesta en {DISTRICT}"

  - id: E-008
    title_es: "Accidente de seguridad"
    category: SEGURIDAD
    type: TARGETED
    severity: LOW
    effects:
      cash: -50000
    headlines:
      - "Guardia herido en {PROPERTY}"

# -----------------------------
# CATEGORY: LICENCIAS (25%)
# -----------------------------

  - id: E-010
    title_es: "Inspección municipal"
    category: LICENCIAS
    type: TARGETED
    severity: LOW
    effects:
      cash: -50000
    mitigation:
      COMPLIANCE_2: { cash: 0 }
    headlines:
      - "Municipio fiscaliza {PROPERTY}"

  - id: E-011
    title_es: "Multa por construcción"
    category: LICENCIAS
    type: TARGETED
    severity: MEDIUM
    condition: "property.upgradeLevel > 0"
    effects:
      cash: -100000
    mitigation:
      COMPLIANCE_2: { cash: -50000 }
    headlines:
      - "{OWNER} multado por obras sin permiso"
      - "Deficiencias estructurales en {PROPERTY}"

  - id: E-012
    title_es: "Licencia vencida"
    category: LICENCIAS
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -100000
      rentDelta: -10000
      rentDuration: 1
    mitigation:
      COMPLIANCE_3: { cash: -50000, rentDelta: 0 }
    headlines:
      - "Clausuran {PROPERTY} temporalmente"

  - id: E-013
    title_es: "Denuncia vecinal"
    category: LICENCIAS
    type: TARGETED
    severity: LOW
    effects:
      cash: -50000
      fame: -1
    headlines:
      - "Vecinos denuncian a {OWNER}"
      - "Quejas por ruido en {PROPERTY}"

  - id: E-014
    title_es: "Revisión de zonificación"
    category: LICENCIAS
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -100000
    headlines:
      - "Municipio cuestiona uso de {PROPERTY}"

  - id: E-015
    title_es: "Deuda tributaria"
    category: LICENCIAS
    type: TARGETED
    severity: HIGH
    effects:
      cash: -150000
    mitigation:
      COMPLIANCE_2: { cash: -100000 }
    headlines:
      - "SUNAT investiga a {OWNER}"
      - "Evasión fiscal en {DISTRICT}?"

  - id: E-016
    title_es: "Embargo preventivo"
    category: LICENCIAS
    type: TARGETED
    severity: HIGH
    effects:
      cash: -100000
      tradeLock: 2
    headlines:
      - "Juez ordena embargo sobre {PROPERTY}"

  - id: E-017
    title_es: "Certificado observado"
    category: LICENCIAS
    type: TARGETED
    severity: LOW
    effects:
      cash: -50000
    headlines:
      - "Defensa Civil observa {PROPERTY}"

  - id: E-018
    title_es: "Conflicto de linderos"
    category: LICENCIAS
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -100000
      tradeLock: 1
    headlines:
      - "Disputa de terrenos afecta a {OWNER}"

  - id: E-019
    title_es: "Auditoría contable"
    category: LICENCIAS
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -100000
    mitigation:
      COMPLIANCE_3: { cash: 0 }
    headlines:
      - "Contraloria revisa cuentas de {OWNER}"

# -----------------------------
# CATEGORY: PRENSA (25%)
# -----------------------------

  - id: E-020
    title_es: "Escándalo mediático"
    category: PRENSA
    type: TARGETED
    severity: HIGH
    effects:
      cash: -50000
      fame: -3
    headlines:
      - "AMPAY: {OWNER} en problemas"
      - "Escándalo sacude a familia {TEAM}"

  - id: E-021
    title_es: "Rumor de quiebra"
    category: PRENSA
    type: TARGETED
    severity: MEDIUM
    effects:
      fame: -2
    headlines:
      - "¿{OWNER} en bancarrota?"
      - "Rumores de crisis financiera en {TEAM}"

  - id: E-022
    title_es: "Denuncia anónima"
    category: PRENSA
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -50000
      fame: -2
    headlines:
      - "Carta anónima acusa a {OWNER}"

  - id: E-023
    title_es: "Investigación periodística"
    category: PRENSA
    type: TARGETED
    severity: HIGH
    effects:
      fame: -3
      politicalExposure: 1
    headlines:
      - "Periodistas destapan conexiones de {OWNER}"
      - "¿Qué oculta la familia {TEAM}?"

  - id: E-024
    title_es: "Entrevista desastrosa"
    category: PRENSA
    type: TARGETED
    severity: MEDIUM
    effects:
      fame: -2
    headlines:
      - "{OWNER} mete la pata en TV"
      - "Declaraciones polémicas de {TEAM}"

  - id: E-025
    title_es: "Foto comprometedora"
    category: PRENSA
    type: TARGETED
    severity: MEDIUM
    effects:
      fame: -2
    headlines:
      - "Imagen viral de {OWNER} causa revuelo"

  - id: E-026
    title_es: "Ex empleado habla"
    category: PRENSA
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -50000
      fame: -2
    headlines:
      - "Ex trabajador revela secretos de {TEAM}"

  - id: E-027
    title_es: "Meme viral"
    category: PRENSA
    type: TARGETED
    severity: LOW
    effects:
      fame: -1
    headlines:
      - "{OWNER} se vuelve meme del día"

  - id: E-028
    title_es: "Comparación desfavorable"
    category: PRENSA
    type: TARGETED
    severity: LOW
    effects:
      fame: -1
    headlines:
      - "¿Por qué {OWNER} no es como los demás?"

  - id: E-029
    title_es: "Crítica de influencer"
    category: PRENSA
    type: TARGETED
    severity: MEDIUM
    effects:
      fame: -2
      rentDelta: -10000
      rentDuration: 1
    headlines:
      - "Tiktoker destroza a {PROPERTY}"

# -----------------------------
# CATEGORY: INCIDENTES (20%)
# -----------------------------

  - id: E-045
    title_es: "Incendio"
    category: INCIDENTES
    type: TARGETED
    severity: HIGH
    effects:
      cash: -150000
      rentDelta: -10000
      rentDuration: 1
      fame: -1
    mitigation:
      INSURANCE_BASIC: { cash: -50000, rentDelta: -5000, fame: 0 }
      INSURANCE_FULL: { cash: 0, rentDelta: -5000, fame: 0 }
    headlines:
      - "Fuego consume parte de {PROPERTY}"
      - "Bomberos controlan incendio en {DISTRICT}"

  - id: E-046
    title_es: "Inundación"
    category: INCIDENTES
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -100000
      rentDelta: -10000
      rentDuration: 1
    mitigation:
      INSURANCE_BASIC: { cash: -50000 }
      INSURANCE_FULL: { cash: 0 }
    headlines:
      - "Agua invade {PROPERTY}"
      - "Rotura de tubería causa estragos"

  - id: E-047
    title_es: "Daño estructural"
    category: INCIDENTES
    type: TARGETED
    severity: HIGH
    effects:
      cash: -150000
      rentDelta: -20000
      rentDuration: 2
    mitigation:
      INSURANCE_BASIC: { cash: -100000 }
      INSURANCE_FULL: { cash: -50000, rentDelta: -10000 }
    headlines:
      - "Grietas peligrosas en {PROPERTY}"
      - "Ingenieros evalúan estructura de {PROPERTY}"

  - id: E-048
    title_es: "Plaga"
    category: INCIDENTES
    type: TARGETED
    severity: LOW
    effects:
      cash: -50000
    mitigation:
      INSURANCE_BASIC: { cash: 0 }
      INSURANCE_FULL: { cash: 0 }
    headlines:
      - "Fumigación urgente en {PROPERTY}"

  - id: E-049
    title_es: "Corte de servicios"
    category: INCIDENTES
    type: TARGETED
    severity: MEDIUM
    effects:
      cash: -50000
      rentDelta: -10000
      rentDuration: 1
    headlines:
      - "Sin luz ni agua en {PROPERTY}"

  - id: E-050
    title_es: "Demanda de inquilino"
    category: INCIDENTES
    type: TARGETED
    severity: HIGH
    effects:
      cash: -150000
      fame: -1
    mitigation:
      INSURANCE_FULL: { cash: 0 }
      COMPLIANCE_2: { fame: 0 }
    headlines:
      - "Inquilino demanda a {OWNER}"
      - "Juicio millonario contra {TEAM}"

  - id: E-051
    title_es: "Expropiación Municipal"
    category: INCIDENTES
    type: TARGETED
    severity: CRITICAL
    condition: DISTRICT_HAS_INFRA
    fallback: E-051B
    effects:
      forceSell: true
      compensationRatio: 1.0
    headlines:
      - "Municipio expropia {PROPERTY} para obra pública"
      - "Adiós a {PROPERTY}: pasa a manos del Estado"

  - id: E-051B
    title_es: "Aviso de derecho de vía"
    category: INCIDENTES
    type: TARGETED
    severity: LOW
    effects:
      cash: -50000
      tradeLock: 2
    headlines:
      - "Municipalidad marca fachada de {PROPERTY} con spray rojo"
      - "Trazo de nueva vía afectaría {PROPERTY}"

  - id: E-052
    title_es: "Accidente de obra"
    category: INCIDENTES
    type: TARGETED
    severity: MEDIUM
    condition: "property.upgradeLevel > 0"
    effects:
      cash: -100000
      fame: -1
    mitigation:
      INSURANCE_BASIC: { cash: -50000 }
      INSURANCE_FULL: { cash: 0, fame: 0 }
    headlines:
      - "Obrero herido en {PROPERTY}"
      - "Investigan accidente en obras de {OWNER}"

  - id: E-053
    title_es: "Anuncio de obras"
    category: INCIDENTES
    type: GLOBAL_TARGET
    severity: NEUTRAL
    logic: ACTIVATE_INFRA
    effects:
      duration: 2
      bias: "50% MarketRow, 50% OwnedMost"
    headlines:
      - "Alcalde promete metro en 2 años (nadie le cree)"
      - "Cierran avenidas por obras"
      - "Nueva vía expresa conectará {DISTRICT}"

# -----------------------------
# CATEGORY: POSITIVE EVENTS
# -----------------------------

  - id: E-P01
    title_es: "Nota favorable"
    category: PRENSA
    type: TARGETED_POSITIVE
    severity: LOW
    effects:
      fame: 1
    headlines:
      - "Elogian gestión de {OWNER}"
      - "{TEAM} destaca en ranking empresarial"

  - id: E-P02
    title_es: "Premio municipal"
    category: LICENCIAS
    type: TARGETED_POSITIVE
    severity: LOW
    effects:
      cash: 50000
      compliance: 1
    headlines:
      - "{PROPERTY} recibe reconocimiento"

  - id: E-P03
    title_es: "Turismo récord"
    category: INCIDENTES
    type: TARGETED_POSITIVE
    severity: MEDIUM
    condition: "property.tags.includes('turismo')"
    effects:
      rentDelta: 20000
      rentDuration: 1
    headlines:
      - "Boom turístico beneficia a {DISTRICT}"
```

## B2: Infrastructure Modifiers

| ID | Name (Spanish) | Effect |
|----|----------------|--------|
| INF-01 | Terminal del Metro anunciado | +GD 5,000 rent, +GD 100,000 worth, enables expropriation |
| INF-02 | Nueva vía expresa | +GD 5,000 rent, +GD 100,000 worth, enables expropriation |

**Duration**: 2 rounds
**Activation**: Via E-053 "Anuncio de obras"

## B3: Global Shocks (Capped)

```yaml
globalShocks:
  - id: G-01
    title_es: "Terremoto"
    effects:
      allProperties:
        rentDelta: -10000
        rentDuration: 2
      allOwners:
        cash: -50000
    headlines:
      - "Temblor sacude Lima"
      - "Pánico en la capital"

  - id: G-02
    title_es: "Crisis económica"
    effects:
      meters:
        Inflation: 1
        InterestRates: 1
      allOwners:
        cash: -100000
    headlines:
      - "Dólar se dispara"
      - "BCR interviene mercado"

  - id: G-03
    title_es: "Escándalo político"
    effects:
      meters:
        PoliticalStability: -1
      weightMods:
        Prensa: 10
    headlines:
      - "Congreso en caos"
      - "Nuevo escándalo en Palacio"

  - id: G-04
    title_es: "Pandemia"
    effects:
      allProperties:
        rentDelta: -20000
        rentDuration: 2
      tags:
        turismo: -20000
    headlines:
      - "Alerta sanitaria nacional"
      - "Cuarentena en Lima"

  - id: G-05
    title_es: "Boom inmobiliario"
    effects:
      allProperties:
        worthDelta: 100000
        worthDuration: 2
    headlines:
      - "Precios de terrenos se disparan"
      - "Inversores extranjeros llegan a Lima"

  - id: G-06
    title_es: "Huelga general"
    effects:
      allOwners:
        cash: -50000
      tags:
        logistica: -20000
    headlines:
      - "Paro nacional paraliza el país"
      - "Carreteras bloqueadas"
```

**Caps by Mode:**
- Quick: 1 per match
- Standard: 2 per match
- Long: 2 per match
- Never more than 1 per 3 rounds

## B4: Insurance Mitigation Tables (Authoritative)

### Incendio (E-045)
| Level | Cash (GD) | Rent (GD) | Fame |
|-------|-----------|-----------|------|
| NONE | -150,000 | -10,000 (1 round) | -1 |
| BASIC | -50,000 | -5,000 (1 round) | 0 |
| FULL | 0 | -5,000 (1 round) | 0 |

### Structural/Liability (E-046, E-047, E-050, E-052)
| Level | Cash Reduction (GD) |
|-------|---------------------|
| BASIC | -50,000 from base penalty |
| FULL | -100,000 from base penalty (+ Fame -1 cancelled) |

### Small Damage (E-048)
| Level | Effect |
|-------|--------|
| BASIC | Cash loss cancelled |
| FULL | Cash loss cancelled |

### Tenant Lawsuit (E-050)
| Level/Condition | Effect |
|-----------------|--------|
| FULL | All cash loss (GD 150,000) cancelled |
| Compliance ≥2 | Fame loss cancelled |

---

# APPENDIX C: UI ACCEPTANCE CRITERIA

## C1: Screen Inventory

| Screen | Key Elements |
|--------|--------------|
| Lobby | Room list, Create button, Join button |
| Room Setup | Mode selector, Team set, Host controls |
| Match HUD | Map, Leaderboard, Timer, Phase indicator |
| Auction | Property cards, Bid panel, Timer |
| Action Bar | Action buttons, Resource display |
| Deal Tray | Incoming deals, Accept/Reject buttons |
| Event Modal | Impact display, Mitigation chips |
| Press Ticker | Headlines, Persona icon |
| End Screen | Portada preview, Share buttons |

## C2: Deal Tray Design (NEW in v10)

**Position**: Right sidebar (desktop) or bottom sheet (mobile)

**States**:
- Empty: "Sin ofertas"
- Has deals: Deal cards stacked

**Deal Card Layout**:
```
┌─────────────────────────────────────┐
│ [From: {OWNER}]           [X Close] │
│ ─────────────────────────────────── │
│ {DEAL_TYPE}                         │
│ {DEAL_DETAILS}                      │
│                                     │
│ [Rechazar]           [Aceptar]      │
└─────────────────────────────────────┘
```

**Animation**: Slide in from right, 200ms ease-out

## C3: Visual Ledger Animation (NEW in v10)

**Trigger**: INCOME_CALC → VISUAL_LEDGER phase transition

**Duration**: 4.0 seconds

**Layout**:
```
┌────────────────────────────────────────┐
│                                        │
│           RESUMEN DE INGRESOS          │
│           ═══════════════════          │
│                                        │
│   Rentas:               +GD 95,000 🟢  │
│   Señales de mercado:    +GD 6,000 🟢  │
│   Bonos de distrito:     +GD 5,000 🟢  │
│   ─────────────────────────────────    │
│   Costos operativos:    -GD 16,000 🔴  │
│   Impuesto a la fama:    -GD 8,000 🔴  │
│   ─────────────────────────────────    │
│                                        │
│   TOTAL:                +GD 82,000     │
│   [████████████████░░░░] → GD 732,000  │
│                                        │
└────────────────────────────────────────┘
```

**Animation Sequence**:
1. (0.0s) Modal appears with header
2. (0.5s) Income lines animate in (stagger 0.2s each)
3. (2.0s) Cost lines animate in (stagger 0.2s each)
4. (3.0s) Total appears with emphasis
5. (3.5s) Cash counter animates to new value
6. (4.0s) Modal fades out, ACTION phase begins

**Constraint**: All input buttons disabled during animation

## C4: Busy Mode Toggle

**Location**: Settings gear in Deal Tray header

**Toggle States**:
- OFF: "Modo normal"
- ON: "Modo Concentrado" (red indicator)

**Effect**: When ON, all incoming deals auto-rejected

---

# APPENDIX D: SHARE ASSETS

## D1: Portada (End-of-Match Share Card)

**Dimensions**: 1200×630 (landscape) + 1080×1920 (stories)

**Layout**:
```
┌────────────────────────────────────────────────────────────┐
│  IMPERIO INMOBILIARIO                    [QR Code]         │
│  ════════════════════                                      │
│                                                            │
│  ┌────────────┐                                            │
│  │  WINNER    │   {WINNER_NAME}                            │
│  │  PORTRAIT  │   Dinastía {TEAM}                          │
│  │            │   Riqueza: ${WEALTH}                       │
│  └────────────┘   Propiedades: {COUNT}                     │
│                                                            │
│  ─────────────────────────────────────────────────────     │
│  "TITULAR 1 DEL MATCH"                                     │
│  "TITULAR 2 DEL MATCH"                                     │
│  "TITULAR 3 DEL MATCH"                                     │
│                                                            │
│  [Sticker 1]  [Sticker 2]  [Sticker 3]                     │
│                                                            │
│  ───────────────────────────────────────────────────────   │
│  Los nombres de dinastías se usan solo como fantasía/      │
│  sátira. No hay afiliación con personas reales.            │
└────────────────────────────────────────────────────────────┘
```

**Required Elements**:
- Masthead "IMPERIO INMOBILIARIO"
- Winner portrait + dynasty crest + color
- 3-5 tabloid headlines + sticker accents
- QR code + short link
- **Mandatory disclaimer footer (Spanish)**

## D2: Trophy Card (Mid-Match Achievement)

**Dimensions**: 1080×1920 (vertical)

**Layout**:
```
┌─────────────────────────────┐
│                             │
│    IMPERIO INMOBILIARIO     │
│                             │
│    ┌─────────────────┐      │
│    │    [STICKER]    │      │
│    │                 │      │
│    │  {ACHIEVEMENT}  │      │
│    │                 │      │
│    └─────────────────┘      │
│                             │
│    {OWNER_NAME}             │
│    Dinastía {TEAM}          │
│                             │
│    "{HEADLINE}"             │
│                             │
│    ────────────────────     │
│    [Disclaimer]             │
└─────────────────────────────┘
```

**Trigger**: Achievement unlock during match

---

# APPENDIX E: AI PORTRAIT PIPELINE

## E1: Portrait Generation Guidelines

**Style**: Cartoon editorial, non-photorealistic

**Prompt Template**:
```
Cartoon portrait avatar for a Peruvian business dynasty member.
Modern flat design, bold outlines, friendly satire style.
Chicha-inspired accent colors. Paper texture background.
NOT based on any real person. Square icon format.
Character type: {DYNASTY_TYPE}
```

**Dynasty Types**:
- Traditional patriarch
- Modern tech entrepreneur
- Old money aristocrat
- Self-made mogul
- Political connected
- Media savvy

## E2: Safety Guidelines

- No identifiable likenesses to real people
- No offensive caricatures
- Diverse representation
- Age-appropriate

---

# APPENDIX F: NETWORKING PROTOCOL

## F1: Envelope Structure

```json
{
  "v": 1,
  "type": "MESSAGE_TYPE",
  "roomId": "RID",
  "matchId": "MID",
  "clientId": "CID",
  "nonce": 123,
  "idempotencyKey": "uuid-v4",
  "payload": {}
}
```

## F2: Message Types

### Client → Server

| Type | Payload | Phase |
|------|---------|-------|
| JOIN_ROOM | { roomId, options } | LOBBY |
| LEAVE_ROOM | { } | Any |
| SET_PROFILE | { handle, team, avatar } | LOBBY |
| SUBMIT_BID | { propertyId, amount, useFameToken? } | BID |
| SUBMIT_ACTION | { actionType, params } | ACTION |
| SUBMIT_DEAL_OFFER | { templateId, toOwnerId, fields } | ACTION |
| SUBMIT_DEAL_RESPONSE | { dealId, response, counterFields? } | ACTION |
| SUBMIT_MOOD_VOTE | { optionId } | MOOD |
| SUBMIT_ELECTION_VOTE | { candidateId, contribution } | VOTE |
| SUBMIT_WATCHER_RUMOR | { targetOwnerId } | Any |
| SUBMIT_WATCHER_BET | { propertyId, predictedWinnerId } | BID |
| SUBMIT_AWARD_VOTE | { awardId, nomineeId } | AWARDS |
| SUBMIT_PACKAGE | { propertyIds, category } | ACTION |
| SUBMIT_QUICK_CHAT | { phraseId, districtParam? } | Any |
| REQUEST_SNAPSHOT | { } | Any |

### Server → Client

| Type | Payload |
|------|---------|
| ROOM_STATE | { participants, settings, status } |
| MATCH_START | { mode, matchId, owners } |
| PHASE_CHANGE | { phase, endsAt, roundSeed? } |
| REVEAL_MARKET | { marketRow, signals } |
| MOOD_OPTIONS_REVEAL | { options } |
| ELECTION_OPTIONS_REVEAL | { candidates } |
| BID_RESULTS | { winners, payments } |
| INCOME_RESULTS | { ownerDeltas } |
| DEALS_UPDATE | { deals } |
| EVENT_RESULTS | { events } |
| PRESS_TICKER | { headlines } |
| ACHIEVEMENT_UNLOCK | { ownerId, achievementId } |
| SHARE_ASSET_READY | { assetType, url } |
| STATE_SNAPSHOT | MatchStateSnapshot |
| MATCH_END | { rankings, portadaUrl } |
| OP_DELTA | { seq, changes } |
| PACKAGE_CREATED | { packageId, ownerId, category, propertyIds } |
| PACKAGE_DISSOLVED | { packageId, reason } |
| QUICK_CHAT_MSG | { fromOwnerId, phraseId, districtParam?, text_es } |
| DEAL_SUGGESTION | { suggestions: Array<{ templateId, prefilled }> } |
| ERROR | { code, message_es } |

## F3: Delta Packets (NEW in v10)

**Purpose**: Reduce bandwidth for frequent state changes

**Structure**:
```json
{
  "type": "OP_DELTA",
  "seq": 105,
  "changes": [
    { "op": "SET_CASH", "id": "O_01", "val": 650000 },
    { "op": "SET_OWNER", "id": "P_05", "val": "O_02" },
    { "op": "SET_FAME", "id": "O_03", "val": 4 },
    { "op": "ADD_PROPERTY", "id": "O_01", "val": "P_05" },
    { "op": "REMOVE_PROPERTY", "id": "O_02", "val": "P_05" }
  ]
}
```

**Operations**:
| Op | Target | Value |
|----|--------|-------|
| SET_CASH | ownerId | number |
| SET_DEBT | ownerId | number |
| SET_FAME | ownerId | number |
| SET_COMPLIANCE | ownerId | 0-3 |
| SET_OWNER | propertyId | ownerId or "" |
| SET_INSURANCE | propertyId | InsuranceLevel |
| SET_UPGRADE | propertyId | 0-2 |
| ADD_PROPERTY | ownerId | propertyId |
| REMOVE_PROPERTY | ownerId | propertyId |
| SET_PHASE | - | Phase |
| SET_ROUND | - | number |
| SET_PACKAGE | propertyId | packageId or "" |
| ADD_PACKAGE | ownerId | packageId |
| REMOVE_PACKAGE | ownerId | packageId |

**Client Handling**:
1. Apply changes in order
2. If `seq` gap detected, request full snapshot
3. Emit `state_changed` event after all changes applied

## F4: Validation Rules

| Rule | Implementation |
|------|----------------|
| Phase check | Reject if wrong phase for message type |
| Nonce check | Reject if nonce ≤ last seen nonce |
| Idempotency | Reject if idempotencyKey already processed |
| Rate limit | Session + IP based limits |
| Ownership check | Verify player owns referenced assets |
| Cash check | Verify player has required funds |

---

# APPENDIX V: VISUAL SPEC (UPDATED v11.2 — "Tycoon Luxury" Style)

## V0: Reference Images

The visual identity is established by the following reference assets:
- **Hero Banner**: Miraflores coastline with "El Magnate" mascot and embossed logo badge
- **Imperio Bill**: Custom $1000 currency with mascot portrait (engraving style)
- **Key Art**: Mascot flying over Lima with raining money

## V1: Art Direction — 3 Pillars (REVISED)

### Pillar 1: "Mobile Tycoon Luxury" Aesthetic
- **Glossy, aspirational mobile game style** (reference: Monopoly GO!, Idle Tycoon games)
- Rich materials: gold, bronze, embossed leather, polished wood
- UI elements with **3D depth, gradients, and subtle shadows**
- Premium feel without being gaudy — sophisticated wealth fantasy

### Pillar 2: "Lima Recognizable" Cityscapes
- **Semi-realistic renderings** of Lima's iconic locations
- Miraflores Costa Verde cliffs as primary backdrop (most recognizable)
- Real landmark silhouettes: Torre Sigma, Larcomar, San Isidro skyline
- Warm lighting, golden hour tones, blue ocean contrast
- NOT flat vector — painterly/illustrated realism

### Pillar 3: "El Magnate" Brand Identity
- **Recurring mascot character**: "El Magnate" (The Tycoon)
  - Elderly distinguished gentleman (70s)
  - White/silver hair, warm smile, confident posture
  - Smoking cigar with visible smoke wisps
  - Navy blue suit with **teal/turquoise tie and pocket square**
  - Gold Rolex-style watch, gold cufflinks
  - NOT based on any real person — generic "wealthy patriarch" archetype
- Mascot appears in: logo, loading screens, currency, achievements, share cards

### Pillar 4: "Imperio Currency" Motif
- **Custom game currency design** styled as US $1000 bill
- Engraving/crosshatch portrait style for mascot face
- Green money tones for currency elements
- "Raining money" particle effects for big wins
- Bills as visual metaphor throughout UI

### Visual Tone Keywords
```
Aspirational, Wealthy, Polished, Lima Pride, Playful Power Fantasy,
Mobile Premium, Sophisticated Humor, Golden Hour, Coastal Luxury
```

### Do-Not-Do (REVISED)
- ❌ Flat vector/minimalist style (too generic)
- ❌ Newspaper/tabloid paper textures (old direction)
- ❌ Chicha poster aesthetics (too local/niche)
- ❌ Cartoon/comic book style (wrong tone)
- ❌ Photorealistic faces of real people
- ❌ Cheap mobile game look (oversaturated, cluttered)
- ❌ Dark/gritty corruption themes (keep it aspirational)

## V2: Logo & Brand Elements

### V2.1 Primary Logo Badge

**Design**:
- Shield/crest shape with dimensional depth
- **Crown** at top (gold with jewel accents)
- **"IMPERIO"** in bold serif, cream/gold gradient with dark outline
- **"INMOBILIARIO"** in smaller caps on red ribbon banner
- **Three gold stars** below ribbon
- Background: deep burgundy red (#8B0000) to dark red gradient
- Border: gold (#FFD700) with bronze (#CD7F32) inner stroke
- Overall embossed/3D medal appearance

**Usage**:
- Splash screen (centered, large)
- Match HUD (top corner, small)
- Share cards (prominent)
- Loading screens

### V2.2 El Magnate Mascot

**Character Design Spec**:
```yaml
age: 70-75 years old
ethnicity: ambiguous/universal (NOT identifiable as specific person)
hair: white/silver, full head, swept back
face: warm smile, kind eyes, slight wrinkles (wisdom not decay)
expression: confident, friendly, successful
pose_default: holding cigar at shoulder height, slight lean
accessories:
  - cigar: thick Cuban-style, visible smoke wisps
  - watch: gold Rolex-style on left wrist
  - cufflinks: gold, visible
  - ring: optional gold signet ring
attire:
  - suit: navy blue, well-tailored, modern fit
  - shirt: white dress shirt
  - tie: teal/turquoise (#008B8B), silk sheen
  - pocket_square: matching teal
```

**Mascot Poses Needed**:
| Pose | Use Case |
|------|----------|
| Standing proud | Logo, splash screen |
| Holding money/bills | Win screens, income |
| Pointing at viewer | Tutorial prompts |
| Thumbs up | Achievement unlocked |
| Worried/sweating | Event damage, losses |
| Riding bill (flying) | Big win celebration |

### V2.3 Imperio Currency ("Graf Dollar Bills")

**Design** — Graf Dollar (GD) denomination bills styled as currency:

| Denomination | Color Accent | Bill Size |
|-------------|-------------|-----------|
| GD 50,000 | Green (standard) | Small bill |
| GD 100,000 | Blue-green | Medium bill |
| GD 500,000 | Gold-accented | Large bill |

**Base Design** (common to all denominations):
- Portrait: El Magnate in engraving/crosshatch style
- Denomination in corners (e.g., "GD 100,000")
- Text: "IMPERIO INMOBILIARIO" as header
- "GRAF DOLLAR" sub-header
- Serial number: decorative (game-generated)
- Color: Traditional green money tones with denomination-specific accents
- Fine filigree borders and ornamental patterns
- "NOT LEGAL TENDER" small print (legal safety)

**Animation Uses**:
- Cash gain: Bills fly toward player (denomination matches amount)
- Cash loss: Bills fly away
- Big win: Raining bills particle effect (GD 500K bills)
- Auction win: Bill stamp/seal animation

**Display Format**: Always show with "GD" prefix and thousands separator: `GD 1,500,000`

## V3: Match UI Kit (Godot Scenes) — REVISED

### V3.1 PropertyCard.tscn

**Visual Style**:
- **Photo-realistic property thumbnail** (Lima locations)
- Gold/bronze frame with embossed edge
- Gradient background (cream to light gold)
- Drop shadow for depth

**Elements**:
- Property photo (top 60%)
- Title banner (nombre_es) with gold text
- District badge (colored chip)
- Price tag: gold coin icon + amount
- Rent display: green bill icon + "/ronda"
- Upgrade stars (0-2, gold when filled)
- Insurance shield icon (bronze/silver/gold by level)

**States**:
- normal: full color
- hovered: golden glow outline
- selected: pulsing gold border
- disabled: desaturated, dark overlay
- sold: "VENDIDO" ribbon across corner (red/gold)

### V3.2 BidPanel.tscn

**Visual Style**:
- Dark wood texture background
- Gold trim and accents
- Embossed buttons

**Elements**:
- Current bid display (large gold numbers)
- +/- stepper buttons (bronze coins style)
- Quick bid buttons: +GD 50K, +GD 100K, +GD 200K (green bills)
- "PUJAR" main button (red with gold border, 3D depth)
- Fame token toggle (star icon, lights up when active)
- Timer: circular countdown with gold ring

### V3.3 ActionBar.tscn

**Visual Style**:
- Horizontal bar at bottom
- Semi-transparent dark background with gold top border
- Icon buttons with text labels below

**Button Design**:
- Circular icons with gold ring border
- Gradient fills (teal for actions, gold for special)
- Subtle glow on hover
- Disabled: grayscale + lock icon overlay

**Buttons**:
| Icon | Label | Color |
|------|-------|-------|
| 🏗️ | Desarrollar | Teal |
| 🛡️ | Asegurar | Blue |
| 👮 | Seguridad | Navy |
| 📢 | PR | Gold |
| 📋 | Auditoría | Gray |
| 🤝 | Trato | Green |
| 💔 | Romper Pacto | Red |
| ⏭️ | Pasar | White |

### V3.4 EventModal.tscn

**Visual Style**:
- Parchment/certificate background texture
- Ornate gold frame border
- El Magnate reaction in corner (worried/relieved based on outcome)

**Elements**:
- Event title (bold, centered)
- Event illustration (semi-realistic scene)
- Impact display:
  - Cash: red bills flying away OR green bills incoming
  - Fame: star icons (+ gold / - gray)
  - Rent: house icon with arrow
- Mitigation badges (insurance shield, security badge)
- "CONTINUAR" button (gold)

### V3.5 PressTicker.tscn

**Visual Style**:
- Breaking news banner style (red/gold)
- Scrolling text with gradient fade edges

**Elements**:
- "NOTICIAS" label with microphone icon
- Headline text (Spanish, scrolling)
- Press persona small avatar

### V3.6 LeaderboardPanel.tscn

**Visual Style**:
- Trophy/podium aesthetic
- Gold/silver/bronze for top 3
- Wood panel background

**Elements**:
- Tab toggle: "RIQUEZA" / "FAMA" (gold tabs)
- Player rows:
  - Rank medal (1st gold, 2nd silver, 3rd bronze, rest numbered)
  - Player avatar thumbnail
  - Dynasty name
  - Value (cash icon or star icon)
- Current player highlighted with glow

### V3.7 DealTray.tscn

**Visual Style**:
- Sliding drawer from right
- Leather briefcase texture
- Gold clasps/trim

**Elements**:
- "OFERTAS" header with count badge
- Deal cards (contract style):
  - Parchment background
  - Wax seal decoration
  - Deal type icon
  - From/To player names
  - Accept (green) / Reject (red) buttons
- "Modo Ocupado" toggle (do not disturb icon)

### V3.8 VisualLedger.tscn

**Visual Style**:
- Bank statement / ledger book aesthetic
- Cream paper with green ruled lines
- Gold embossed "ESTADO DE CUENTA" header

**Elements**:
- Line items with +/- indicators
- Running total animation
- El Magnate thumbnail (happy if positive, concerned if negative)
- Final balance with cash counter animation

## V4: MapView Spec — REVISED

### Map Style
- **Semi-realistic aerial view** of Lima
- Recognizable coastline (Costa Verde cliffs essential)
- Soft painterly rendering (not photographic)
- Golden hour lighting
- Ocean visible on west edge
- Mountains faintly visible on east

### District Rendering
- Districts as **soft-bordered regions** (not hard vector shapes)
- Each district has:
  - Subtle color tint overlay
  - Iconic landmark silhouette
  - District name label (gold text with shadow)

### District Regions (12) with Landmarks
| District | Landmark Reference |
|----------|-------------------|
| Miraflores | Costa Verde cliffs, Larcomar |
| San Isidro | Glass towers, El Olivar park |
| Barranco | Puente de los Suspiros, colorful houses |
| Surco | Jockey Plaza area |
| La Molina | Hills, residential sprawl |
| Centro | Plaza de Armas, Cathedral |
| Jesus Maria/Lince | Campo de Marte |
| Magdalena/San Miguel | Costa Verde north |
| Callao | Port cranes, La Punta |
| Ate/SJL | Industrial zone, hills |
| Chorrillos | Morro Solar, fishing pier |
| Independencia/Los Olivos | MegaPlaza area |

### Property Markers
- **3D building icons** (mini versions of actual property types)
- Dynasty color glow ring around base
- Upgrade level: additional floors/details on building
- Insurance: small shield floating above
- Hover: building "pops up" with info card

### INF Corridor Visualization
- **Metro line**: animated train icon moving along dotted line
- **Via Expresa**: highway with tiny moving cars
- Active corridor: golden glow, "EN CONSTRUCCIÓN" banner
- Expropriation: red warning pulse, demolition crane icon

## V5: Motion & Microinteractions — REVISED

### Durations
| Type | Duration |
|------|----------|
| UI transitions | 200-300ms (easeOutQuart) |
| Big moments | 500-800ms |
| Particle effects | 1-2s |
| Timer warning | Golden pulse every 1s (last 5s) |

### Signature Animations
| Event | Animation |
|-------|-----------|
| Auction win | Gold confetti burst + "VENDIDO" ribbon stamp + El Magnate thumbs up |
| Cash gain | Bills flying toward player + counter tick up + ka-ching sound |
| Cash loss | Bills flying away + counter tick down + El Magnate worried face |
| Achievement | Trophy/badge 3D spin + golden rays + confetti |
| Big win | Raining money (bills falling) + El Magnate riding bill across screen |
| Event damage | Screen shake + red vignette flash + impact dust |
| Level up | Golden light pillar + sparkle particles |
| Timer critical | Red pulse + heartbeat sound + screen edge glow |

### Particle Systems
- **Money rain**: 20-50 bill sprites falling with rotation
- **Confetti**: Gold and teal paper pieces (50-100 sprites)
- **Sparkles**: Small star particles for highlights
- **Smoke**: Cigar smoke wisps for mascot
- **Dust/debris**: For destruction/construction events

## V6: Audio — REVISED

### SFX (12 Required)
| ID | Sound | Description |
|----|-------|-------------|
| sfx_bid | Bid submit | Poker chip click |
| sfx_auction_win | Auction won | Gavel slam + crowd cheer |
| sfx_cash_gain | Money received | Ka-ching register + coins |
| sfx_cash_loss | Money lost | Low orchestral sting + paper shuffle (bills departing) |
| sfx_stamp | Official action | Heavy stamp thud |
| sfx_deal_accept | Deal completed | Handshake + paper shuffle |
| sfx_deal_reject | Deal rejected | Paper tear |
| sfx_achievement | Badge unlocked | Triumphant fanfare (short) |
| sfx_timer_tick | Timer countdown | Clock tick (last 10s) |
| sfx_timer_urgent | Timer critical | Heartbeat (last 5s) |
| sfx_event_bad | Negative event | Dramatic sting |
| sfx_event_good | Positive event | Uplifting chime |

### Music (3 Tracks)
| Track | Use | Style |
|-------|-----|-------|
| music_menu | Lobby/menus | Smooth jazz, sophisticated |
| music_match | During gameplay | Upbeat, suspenseful, Latin jazz influence |
| music_victory | End of match | Triumphant orchestral, celebratory |

### Voice Stingers (Optional, Spanish)
- "¡VENDIDO!" (Auction win)
- "¡FELICIDADES!" (Achievement)
- "¡AL TOQUE!" (Quick action)
- "¡QUÉ JUGADA!" (Big move)
- "¡OJO!" (Warning/event)

## V7: Typography & Colors — REVISED

### Typography
| Context | Font | Weight | Use |
|---------|------|--------|-----|
| Logo "IMPERIO" | Custom serif (Cinzel or Trajan Pro) | Bold | Brand |
| Headlines | Playfair Display | Bold | Event titles, announcements |
| UI Labels | Montserrat | SemiBold | Buttons, labels |
| Body text | Open Sans | Regular | Descriptions, details |
| Numbers/Money | Oswald | Bold | Cash displays, stats |

### Color Palette — "Tycoon Luxury"

**Primary Colors**:
| Name | Hex | Use |
|------|-----|-----|
| Imperio Gold | #FFD700 | Primary accent, highlights |
| Imperio Bronze | #CD7F32 | Secondary accent, borders |
| Imperio Red | #8B0000 | Logo background, warnings |
| Teal Accent | #008B8B | Tie color, action buttons |

**Neutral Colors**:
| Name | Hex | Use |
|------|-----|-----|
| Rich Navy | #1B1B3A | Suit color, dark backgrounds |
| Cream | #FFFDD0 | Light backgrounds, paper |
| Warm White | #FAF9F6 | Text on dark |
| Charcoal | #333333 | Primary text |

**Functional Colors**:
| Name | Hex | Use |
|------|-----|-----|
| Money Green | #228B22 | Positive cash, currency |
| Loss Red | #DC143C | Negative events, cash loss |
| Fame Gold | #FFB347 | Fame points, stars |
| Info Blue | #4169E1 | Information, links |
| Sky Blue | #87CEEB | Lima sky, backgrounds |

**Gradient Presets**:
```css
/* Logo badge background */
linear-gradient(180deg, #B22222 0%, #8B0000 100%)

/* Gold button */
linear-gradient(180deg, #FFD700 0%, #CD7F32 100%)

/* Sky background */
linear-gradient(180deg, #87CEEB 0%, #E0F4FF 100%)
```

## V8: Asset Inventory (MVP) — REVISED

| Asset Type | Count | Format |
|------------|-------|--------|
| **Brand Assets** | | |
| Logo badge (various sizes) | 5 | PNG (transparent) |
| El Magnate mascot poses | 8 | PNG (transparent) |
| Imperio Bill design | 3 | PNG |
| **Backgrounds** | | |
| Lima cityscape (Miraflores hero) | 1 | PNG 1920x1080 |
| District backdrops | 12 | PNG 1920x1080 |
| Sky gradients | 3 | PNG tileable |
| **UI Elements** | | |
| Property card frames | 4 | PNG (9-slice) |
| Button styles | 6 | PNG (9-slice) |
| Icon set (actions/status) | 40 | PNG 128x128 |
| Achievement badges | 20 | PNG 256x256 |
| **Map Assets** | | |
| Lima aerial map base | 1 | PNG 2048x2048 |
| District overlays | 12 | PNG (transparent) |
| Property building icons | 15 | PNG 128x128 |
| Corridor overlays (metro/via) | 2 | PNG (animated) |
| **Effects** | | |
| Bill sprite (for particles) | 1 | PNG 64x128 |
| Confetti sprites | 5 | PNG 32x32 |
| Sparkle sprites | 3 | PNG 32x32 |

## V9: AI Generation Prompts — REVISED (Tycoon Style)

### Hero Cityscape (Miraflores)
```
Aerial view of Miraflores district Lima Peru, Costa Verde cliffs and coastline,
modern glass skyscrapers, Pacific Ocean, golden hour lighting, clear blue sky,
semi-realistic digital painting style, mobile game background art,
aspirational wealthy vibe, no people visible, cinematic composition,
warm tones, professional architectural visualization style --ar 16:9
```

### El Magnate Mascot (Base Pose)
```
Distinguished elderly businessman character, 70 years old, white silver hair
swept back, warm friendly smile, smoking Cuban cigar with visible smoke wisps,
wearing navy blue tailored suit with teal turquoise tie and pocket square,
gold Rolex watch visible, gold cufflinks, confident successful pose,
caricature portrait style like mobile tycoon game mascot,
NOT based on any real person, generic wealthy patriarch archetype,
clean white background, full body or waist up --ar 1:1
```

### Imperio Bill Currency
```
Ornate currency bill design, $1000 denomination, portrait style placeholder
in center oval frame, fine line engraving crosshatch patterns,
decorative filigree borders, green money color scheme,
"IMPERIO INMOBILIARIO" as header text, serial numbers in corners,
vintage US dollar bill aesthetic, highly detailed ornamental design,
NOT legal tender, game currency art --ar 16:9
```

### District Backdrops (Template)
```
Aerial view of {DISTRICT} district in Lima Peru, {LANDMARK_DESCRIPTION},
golden hour lighting, semi-realistic digital painting, mobile game background,
clear blue sky, warm aspirational tones, no people,
architectural visualization style, cinematic depth --ar 16:9
```

### Property Card Thumbnails
```
{PROPERTY_TYPE} building in Lima Peru, {DISTRICT} district style architecture,
semi-realistic rendering, golden hour lighting, front facade view,
mobile game property card art, clean composition, slight upward angle,
aspirational real estate photography style --ar 4:3
```

### Achievement Badge
```
Luxury game achievement badge, {ACHIEVEMENT_THEME}, circular medal design,
embossed gold and bronze metallic finish, 3D depth and shadows,
rich red velvet or navy blue accent, ornate decorative border,
mobile game reward icon style, premium feel --ar 1:1
```

---

# APPENDIX O: OPEN SOURCE FOUNDATIONS (NEW in v11)

## O1: Server Framework Evaluation

| Repository | Stars | License | Pros | Cons | Fit |
|------------|-------|---------|------|------|-----|
| [colyseus/colyseus](https://github.com/colyseus/colyseus) | 5k+ | MIT | Authoritative state, rooms, TypeScript native, delta sync | Smaller community than Nakama | ⭐ **BEST** |
| [heroiclabs/nakama](https://github.com/heroiclabs/nakama) | 9k+ | Apache 2.0 | Matchmaking, leaderboards, more infra | Go-based, heavier setup | Good |
| [boardgameio/boardgame.io](https://github.com/boardgameio/boardgame.io) | 10k+ | MIT | Turn-based focused, good abstractions | React-centric, less flexible | Good |

**Recommendation**: Colyseus for core server

### Why Colyseus
1. **Authoritative State**: Built-in state synchronization with schema validation
2. **Room Architecture**: Matches our game room concept exactly
3. **TypeScript Native**: Same language as our server codebase
4. **Delta Compression**: Efficient state updates out of the box
5. **WebSocket Handling**: Battle-tested reconnection and session management

## O2: Client Framework Evaluation

| Repository | License | Pros | Cons | Fit |
|------------|---------|------|------|-----|
| [godotengine/godot](https://github.com/godotengine/godot) | MIT | Full engine, Web export, active community | Learning curve | ⭐ **PRIMARY** |
| [GodotJS](https://godotjs.github.io/) | MIT | TypeScript in Godot | Early stage | Good addon |
| [db0/godot-card-game-framework](https://github.com/db0/godot-card-game-framework) | AGPL3 | Card UI components | AGPL license | Reference only |
| [chun92/card-framework](https://github.com/chun92/card-framework) | MIT | Godot 4.x cards | Limited features | Good starting point |

### Useful References

| Repository | Use For |
|------------|---------|
| [AnicetNgrt/GodotOnlineRoomSystem](https://github.com/AnicetNgrt/GodotOnlineRoomSystem) | Room boilerplate |
| [Connorvangraan/PropertyTycoon](https://github.com/Connorvangraan/PropertyTycoon) | Monopoly-like mechanics |
| [gamescomputersplay/monopoly](https://github.com/gamescomputersplay/monopoly) | Simulation logic |

## O3: License Compatibility

| License | Commercial Use | Modifications | Distribution | Patent Grant |
|---------|----------------|---------------|--------------|--------------|
| MIT | ✅ | ✅ | ✅ | ❌ |
| Apache 2.0 | ✅ | ✅ | ✅ | ✅ |
| AGPL3 | ⚠️ (copyleft) | ✅ | ⚠️ (source required) | ✅ |

**Guidance**:
- Prefer MIT/Apache 2.0 for code you'll modify
- AGPL3 is fine for reference/inspiration only
- Always include license notices in distribution

## O4: Integration Code Examples

### Colyseus Room Setup (Server)

```typescript
// server/src/rooms/ImperioRoom.ts
import { Room, Client, Delayed } from "colyseus";
import { ImperioState, OwnerSchema, PropertySchema } from "./schema";

export class ImperioRoom extends Room<ImperioState> {
  maxClients = 20;

  // Phase timing
  private phaseTimer: Delayed;

  onCreate(options: { mode: "QUICK" | "STANDARD" | "LONG" }) {
    this.setState(new ImperioState());
    this.state.mode = options.mode;

    // Configure based on mode
    const config = this.getModeConfig(options.mode);
    this.state.maxRounds = config.rounds;

    // Set up message handlers
    this.onMessage("bid", this.handleBid.bind(this));
    this.onMessage("action", this.handleAction.bind(this));
    this.onMessage("deal_offer", this.handleDealOffer.bind(this));
    this.onMessage("deal_response", this.handleDealResponse.bind(this));
    this.onMessage("mood_vote", this.handleMoodVote.bind(this));
    this.onMessage("election_vote", this.handleElectionVote.bind(this));

    // Start match when ready
    this.onMessage("start_match", this.startMatch.bind(this));
  }

  onJoin(client: Client, options: any) {
    if (this.state.owners.size < 12) {
      // Add as Owner
      const owner = new OwnerSchema();
      owner.ownerId = client.sessionId;
      owner.handle = options.handle || `Player${this.state.owners.size + 1}`;
      owner.team = options.team || this.getRandomTeam();
      owner.cash = this.getStartingCash();
      this.state.owners.set(client.sessionId, owner);
    } else {
      // Add as Watcher
      this.state.watcherCount++;
    }
  }

  async onLeave(client: Client, consented: boolean) {
    const owner = this.state.owners.get(client.sessionId);
    if (owner) {
      if (!consented) {
        // Allow reconnection for 60 seconds
        try {
          await this.allowReconnection(client, 60);
          return;
        } catch (e) {
          // Reconnection failed, enable autopilot
          owner.isBot = true;
        }
      } else {
        this.state.owners.delete(client.sessionId);
      }
    } else {
      this.state.watcherCount--;
    }
  }

  private handleBid(client: Client, data: { propertyId: string; amount: number; useFameToken?: boolean }) {
    if (this.state.phase !== "BID") {
      client.send("error", { code: "WRONG_PHASE", message_es: "No es momento de pujar" });
      return;
    }

    const owner = this.state.owners.get(client.sessionId);
    if (!owner) return;

    if (data.amount > owner.cash) {
      client.send("error", { code: "INSUFFICIENT_CASH", message_es: "No te alcanza, causa." });
      return;
    }

    // Record bid (resolve at phase end)
    this.state.pendingBids.set(client.sessionId, {
      propertyId: data.propertyId,
      amount: data.amount,
      useFameToken: data.useFameToken || false
    });
  }

  private transitionToPhase(phase: string, duration: number) {
    this.state.phase = phase;
    this.state.phaseEndsAt = Date.now() + duration;

    // Broadcast phase change
    this.broadcast("phase_change", {
      phase,
      endsAt: this.state.phaseEndsAt,
      roundSeed: phase === "REVEAL" ? this.state.roundSeed : undefined
    });

    // Schedule next phase
    this.phaseTimer = this.clock.setTimeout(() => {
      this.processPhaseEnd(phase);
    }, duration);
  }

  private getModeConfig(mode: string) {
    const configs = {
      QUICK: { rounds: 7, bidTime: 25000, actionTime: 25000 },
      STANDARD: { rounds: 10, bidTime: 35000, actionTime: 35000 },
      LONG: { rounds: 14, bidTime: 40000, actionTime: 40000 }
    };
    return configs[mode] || configs.STANDARD;
  }
}
```

### Godot Client Connection (GDScript)

```gdscript
# client/scripts/network/ImperioClient.gd
extends Node

class_name ImperioClient

signal connected
signal disconnected
signal phase_changed(phase: String, ends_at: int)
signal state_updated(state: Dictionary)
signal error_received(code: String, message: String)

var websocket: WebSocketPeer
var server_url: String = "ws://localhost:2567"
var room_id: String = ""
var session_id: String = ""
var last_seq: int = 0

func _ready():
    websocket = WebSocketPeer.new()

func _process(_delta):
    websocket.poll()

    var state = websocket.get_ready_state()
    if state == WebSocketPeer.STATE_OPEN:
        while websocket.get_available_packet_count():
            _handle_packet(websocket.get_packet().get_string_from_utf8())
    elif state == WebSocketPeer.STATE_CLOSED:
        emit_signal("disconnected")

func join_room(room: String, options: Dictionary = {}):
    room_id = room
    var url = "%s/%s" % [server_url, room]
    var err = websocket.connect_to_url(url)
    if err != OK:
        push_error("Failed to connect: %d" % err)
        return

    # Send join message once connected
    await get_tree().create_timer(0.5).timeout
    _send({
        "type": "join",
        "options": options
    })

func submit_bid(property_id: String, amount: int, use_fame_token: bool = false):
    _send({
        "type": "bid",
        "propertyId": property_id,
        "amount": amount,
        "useFameToken": use_fame_token
    })

func submit_action(action_type: String, params: Dictionary = {}):
    _send({
        "type": "action",
        "actionType": action_type,
        "params": params
    })

func submit_deal_offer(template_id: String, to_owner_id: String, fields: Dictionary):
    _send({
        "type": "deal_offer",
        "templateId": template_id,
        "toOwnerId": to_owner_id,
        "fields": fields
    })

func submit_deal_response(deal_id: String, accept: bool, counter_fields: Dictionary = {}):
    _send({
        "type": "deal_response",
        "dealId": deal_id,
        "response": "ACCEPTED" if accept else "REJECTED",
        "counterFields": counter_fields
    })

func _send(data: Dictionary):
    var json = JSON.stringify(data)
    websocket.send_text(json)

func _handle_packet(data: String):
    var json = JSON.parse_string(data)
    if json == null:
        push_error("Invalid JSON: %s" % data)
        return

    match json.get("type", ""):
        "joined":
            session_id = json.sessionId
            emit_signal("connected")

        "phase_change":
            emit_signal("phase_changed", json.phase, json.endsAt)

        "state_snapshot":
            emit_signal("state_updated", json.payload)

        "OP_DELTA":
            _apply_delta(json)

        "error":
            emit_signal("error_received", json.code, json.message_es)

func _apply_delta(delta: Dictionary):
    if delta.seq <= last_seq:
        return  # Ignore old deltas

    if delta.seq > last_seq + 1:
        # Gap detected, request full snapshot
        _send({"type": "request_snapshot"})
        return

    last_seq = delta.seq

    # Apply changes (implementation depends on your state structure)
    for change in delta.changes:
        _apply_change(change)

    emit_signal("state_updated", {})  # Trigger UI refresh

func _apply_change(change: Dictionary):
    # Override in subclass or use signals
    pass
```

### Visual Ledger Animation (GDScript)

```gdscript
# client/scripts/ui/VisualLedger.gd
extends Control

class_name VisualLedger

@onready var income_container: VBoxContainer = $Panel/IncomeContainer
@onready var costs_container: VBoxContainer = $Panel/CostsContainer
@onready var total_label: Label = $Panel/TotalLabel
@onready var cash_counter: Label = $Panel/CashCounter
@onready var animation_player: AnimationPlayer = $AnimationPlayer

var income_items: Array[Dictionary] = []
var cost_items: Array[Dictionary] = []
var total_delta: int = 0
var starting_cash: int = 0
var final_cash: int = 0

signal animation_complete

func show_ledger(data: Dictionary):
    # Parse income data
    income_items = data.get("income", [])
    cost_items = data.get("costs", [])
    total_delta = data.get("total", 0)
    starting_cash = data.get("startingCash", 0)
    final_cash = data.get("finalCash", 0)

    # Clear containers
    _clear_container(income_container)
    _clear_container(costs_container)

    # Show panel
    visible = true
    modulate.a = 0

    # Start animation sequence
    _animate_sequence()

func _animate_sequence():
    # Fade in
    var tween = create_tween()
    tween.tween_property(self, "modulate:a", 1.0, 0.3)
    await tween.finished

    # Animate income items (staggered)
    for item in income_items:
        await _add_income_item(item)
        await get_tree().create_timer(0.2).timeout

    await get_tree().create_timer(0.3).timeout

    # Animate cost items (staggered)
    for item in cost_items:
        await _add_cost_item(item)
        await get_tree().create_timer(0.2).timeout

    await get_tree().create_timer(0.3).timeout

    # Show total with emphasis
    _show_total()
    await get_tree().create_timer(0.5).timeout

    # Animate cash counter
    await _animate_cash_counter()

    await get_tree().create_timer(0.5).timeout

    # Fade out
    var fade_out = create_tween()
    fade_out.tween_property(self, "modulate:a", 0.0, 0.3)
    await fade_out.finished

    visible = false
    emit_signal("animation_complete")

func _add_income_item(item: Dictionary):
    var label = Label.new()
    label.text = "%s: +$%d" % [item.name, item.amount]
    label.add_theme_color_override("font_color", Color.GREEN)
    label.modulate.a = 0
    income_container.add_child(label)

    var tween = create_tween()
    tween.tween_property(label, "modulate:a", 1.0, 0.15)

func _add_cost_item(item: Dictionary):
    var label = Label.new()
    label.text = "%s: -$%d" % [item.name, item.amount]
    label.add_theme_color_override("font_color", Color.RED)
    label.modulate.a = 0
    costs_container.add_child(label)

    var tween = create_tween()
    tween.tween_property(label, "modulate:a", 1.0, 0.15)

func _show_total():
    var color = Color.GREEN if total_delta >= 0 else Color.RED
    var sign = "+" if total_delta >= 0 else ""
    total_label.text = "TOTAL: %s$%d" % [sign, total_delta]
    total_label.add_theme_color_override("font_color", color)

    # Scale punch effect
    total_label.scale = Vector2(1.3, 1.3)
    var tween = create_tween()
    tween.tween_property(total_label, "scale", Vector2.ONE, 0.2).set_ease(Tween.EASE_OUT)

func _animate_cash_counter():
    var tween = create_tween()
    tween.tween_method(_update_cash_display, starting_cash, final_cash, 0.5)

func _update_cash_display(value: int):
    cash_counter.text = "$%d" % value

func _clear_container(container: Container):
    for child in container.get_children():
        child.queue_free()
```

---

# APPENDIX G: AI ASSET GENERATION PIPELINE (NEW in v11)

This appendix provides specific AI services and prompts for generating all visual assets required by the game. Following these guidelines ensures visual consistency across the entire asset library.

## G1: Service Selection Matrix — REVISED for "Tycoon Luxury" Style

### G1.1 Recommended Services by Asset Type

| Asset Type | Primary Service | Alternative | Output Format | Est. Cost/Asset |
|------------|-----------------|-------------|---------------|-----------------|
| Hero Cityscape (Miraflores) | Midjourney v6.1 | Flux Pro | PNG 16:9 | $0.08-0.16 |
| District Backdrops | Midjourney v6.1 | Leonardo.ai | PNG 16:9 | $0.04-0.08 |
| El Magnate Mascot | Midjourney v6.1 | Leonardo.ai | PNG (transparent) | $0.08-0.16 |
| Imperio Bill Currency | Midjourney v6.1 | DALL-E 3 | PNG 16:9 | $0.04-0.08 |
| Logo Badge | Ideogram 2.0 | Midjourney v6.1 | PNG (transparent) | $0.04-0.08 |
| Property Photos | Midjourney v6.1 | Flux Pro | PNG 4:3 | $0.04-0.08 |
| Achievement Badges | Midjourney v6.1 | Leonardo.ai | PNG 1:1 | $0.04-0.08 |
| UI Elements/Frames | Midjourney v6.1 | DALL-E 3 | PNG (9-slice) | $0.04-0.08 |
| Dynasty Crests | Midjourney v6.1 | Ideogram 2.0 | PNG 1:1 | $0.04-0.08 |
| Map Aerial View | Midjourney v6.1 | Flux Pro | PNG 1:1 | $0.08-0.16 |

### G1.2 Service Comparison — For Tycoon Style

| Service | Strengths | Weaknesses | Best For |
|---------|-----------|------------|----------|
| **Midjourney v6.1** | Best for painterly realism, excellent architecture, consistent characters | No API, Discord only | Hero assets, cityscapes, mascot |
| **Flux Pro (via Replicate)** | Excellent photorealism, API access, fast | Less stylized control | Architectural renders, backdrops |
| **Leonardo.ai Phoenix** | Good characters, fine control, API | Can be inconsistent | Character variations, UI assets |
| **DALL-E 3** | API access, good composition | Less cinematic quality | Quick iterations, UI elements |
| **Ideogram 2.0** | Excellent text rendering, logos | Limited painterly styles | Logo badge, text-heavy graphics |
| **Stable Diffusion XL + ControlNet** | Free, customizable, inpainting | Setup required | Batch work, refinements |

### G1.3 Workflow Recommendation

```
1. HERO ASSETS (Midjourney v6.1)
   - Miraflores cityscape
   - El Magnate mascot (all poses)
   - Imperio Bill design

2. ARCHITECTURAL RENDERS (Midjourney or Flux)
   - 12 district backdrops
   - Property card thumbnails

3. UI/BADGES (Midjourney + Ideogram)
   - Achievement badges
   - Dynasty crests
   - Logo badge refinement

4. POST-PROCESSING (Photoshop/Figma)
   - Background removal
   - Color consistency
   - Final composition
```

## G2: Style Guide — "Tycoon Luxury" Aesthetic

### G2.1 Core Style Keywords (Use in ALL prompts)

```
STYLE ANCHOR (include in EVERY prompt):
"Semi-realistic digital painting, mobile tycoon game art style,
golden hour warm lighting, aspirational wealthy aesthetic,
cinematic composition, professional architectural visualization,
soft painterly rendering, premium mobile game quality"
```

### G2.2 Color Palette Reference

```
PRIMARY COLORS (include in prompts):
- Imperio Gold: #FFD700 (highlights, accents)
- Imperio Bronze: #CD7F32 (borders, secondary)
- Imperio Red: #8B0000 (logo, warnings)
- Teal Accent: #008B8B (signature color, mascot tie)
- Rich Navy: #1B1B3A (dark backgrounds, suit)

ENVIRONMENTAL COLORS:
- Sky Blue: #87CEEB (Lima sky)
- Ocean Blue: #4682B4 (Pacific Ocean)
- Cliff Brown: #8B7355 (Costa Verde cliffs)
- Grass Green: #228B22 (parks, lawns)
- Concrete Gray: #A9A9A9 (buildings, roads)

MONEY/CURRENCY:
- Bill Green: #228B22 (currency)
- Gold Coin: #FFD700 (wealth indicators)
```

### G2.3 Character Reference — El Magnate

```
MASCOT DESCRIPTION (include for character consistency):
"Distinguished elderly businessman, 70 years old, Caucasian/ambiguous ethnicity,
white silver hair swept back neatly, warm friendly confident smile,
smoking thick Cuban cigar with visible smoke wisps,
wearing perfectly tailored navy blue suit, white dress shirt,
teal turquoise silk tie and matching pocket square,
gold Rolex-style watch on left wrist, gold cufflinks,
caricature style like Monopoly man meets mobile game tycoon,
NOT based on any real person, generic wealthy patriarch archetype"
```

### G2.4 Negative Prompts (Exclude from results)

```
UNIVERSAL NEGATIVE PROMPT:
"flat vector, cartoon, comic book, anime, chibi, watercolor,
sketch, pencil drawing, minimalist, abstract, ugly, deformed,
bad anatomy, extra limbs, bad hands, blurry, low quality,
watermark, signature, text overlay, stock photo, generic,
identifiable real person, celebrity likeness, offensive,
dark gritty, dystopian, poverty, corruption imagery"
```

### G2.5 Lima Location References

When generating Lima cityscapes, reference these real landmarks:
```
MIRAFLORES:
- Costa Verde cliffs (green clifftop, brown cliffs, beach below)
- Torre Sigma (twisted glass skyscraper)
- Larcomar (cliffside shopping center)
- Parque Kennedy (central park)
- Marriott hotel (beachfront tower)

SAN ISIDRO:
- Financial district glass towers
- El Olivar park (olive trees)
- Country Club Lima

BARRANCO:
- Puente de los Suspiros (Bridge of Sighs)
- Colorful colonial houses
- Bajada de Baños

CALLAO:
- Port cranes and containers
- La Punta peninsula
- Real Felipe fortress
```

## G3: Asset-Specific Prompts — TYCOON LUXURY STYLE

### G3.1 Hero Cityscape — Miraflores (Primary Background)

**Service**: Midjourney v6.1
**Aspect Ratio**: --ar 16:9
**Style**: --s 250 --style raw

```
HERO PROMPT (Miraflores - Main Background):
"Stunning aerial view of Miraflores district Lima Peru, Costa Verde green clifftops
and dramatic brown coastal cliffs, Pacific Ocean turquoise waters, modern glass
skyscrapers including twisted Torre Sigma tower, Larcomar shopping center on cliffs,
Costa Verde highway with tiny cars, golden hour warm sunlight, clear blue sky
with few white clouds, semi-realistic digital painting, mobile tycoon game
background art, aspirational wealthy coastal city aesthetic, cinematic wide shot,
architectural visualization quality, no people visible --ar 16:9 --s 250 --style raw"
```

**Variations**:
```yaml
hero_variations:
  - time: "golden_hour"
    suffix: "golden hour sunset lighting, warm orange and pink sky tones"

  - time: "midday"
    suffix: "bright midday sun, vivid blue sky, high contrast shadows"

  - time: "evening"
    suffix: "blue hour twilight, city lights beginning to glow, purple sky"
```

### G3.2 District Backdrops (12 Districts)

**Service**: Midjourney v6.1
**Aspect Ratio**: --ar 16:9
**Style**: --s 200 --style raw

```
TEMPLATE PROMPT:
"Aerial view of {DISTRICT} district in Lima Peru, {LANDMARK_DESCRIPTION},
golden hour warm lighting, semi-realistic digital painting style,
mobile tycoon game background art, aspirational aesthetic,
clear sky, cinematic composition, architectural visualization,
no people visible, no text --ar 16:9 --s 200 --style raw"
```

**District-Specific Prompts**:

```yaml
districts:
  - name: Miraflores
    landmarks: "Costa Verde cliffs, beachfront hotels, Larcomar mall, Pacific Ocean"
    mood: "luxury coastal, modern wealth"

  - name: San Isidro
    landmarks: "glass skyscraper financial district, El Olivar olive tree park, corporate towers"
    mood: "business district, corporate power"

  - name: Barranco
    landmarks: "colorful colonial houses, Puente de los Suspiros bridge, bohemian streets"
    mood: "artistic, charming, vintage luxury"

  - name: Surco
    landmarks: "Jockey Plaza shopping center, residential neighborhoods, tree-lined streets"
    mood: "upscale suburban, family wealth"

  - name: La Molina
    landmarks: "hillside mansions, gated communities, green hills, eucalyptus trees"
    mood: "exclusive, old money estates"

  - name: Centro
    landmarks: "Plaza de Armas, colonial cathedral, Government Palace, historic buildings"
    mood: "historic power, colonial grandeur"

  - name: Jesus Maria/Lince
    landmarks: "Campo de Marte park, mid-rise apartments, commercial avenues"
    mood: "middle class aspiration"

  - name: Magdalena/San Miguel
    landmarks: "Costa Verde northern stretch, university buildings, mixed residential"
    mood: "coastal urban, emerging"

  - name: Callao
    landmarks: "port with shipping cranes and containers, La Punta peninsula, naval presence"
    mood: "industrial power, trade wealth"

  - name: Ate/SJL
    landmarks: "industrial warehouses, commercial zones, Andes foothills visible"
    mood: "working wealth, commerce"

  - name: Chorrillos
    landmarks: "Morro Solar hill, fishing pier, beach town, modest but charming"
    mood: "traditional coastal, authentic"

  - name: Independencia/Los Olivos
    landmarks: "MegaPlaza commercial center, busy avenues, emerging skyline"
    mood: "new money, commercial growth"
```

### G3.3 El Magnate Mascot (8 Poses)

**Service**: Midjourney v6.1
**Aspect Ratio**: --ar 1:1 (square) or --ar 2:3 (portrait)
**Style**: --s 150 --style raw

```
BASE CHARACTER PROMPT:
"Distinguished elderly businessman character, 70 years old, white silver hair
swept back elegantly, warm confident smile with slight smirk, holding thick
Cuban cigar with visible smoke wisps near shoulder, wearing perfectly tailored
navy blue suit jacket, white dress shirt, teal turquoise silk tie and matching
pocket square, gold Rolex watch visible on left wrist, gold cufflinks,
caricature portrait style like premium mobile tycoon game mascot character,
NOT based on any real person, generic successful patriarch archetype,
{POSE_DESCRIPTION}, clean background or transparent,
semi-realistic digital painting --ar 1:1 --s 150 --style raw"
```

**Pose Variations**:

```yaml
mascot_poses:
  - id: standing_proud
    pose: "standing confidently with slight forward lean, one hand on lapel"
    use: "Logo, splash screen, profile"
    aspect: "2:3"

  - id: holding_money
    pose: "holding a stack of cash bills fanned out, pleased expression"
    use: "Win screens, income phase"
    aspect: "1:1"

  - id: pointing
    pose: "pointing confidently at viewer with cigar hand, inviting expression"
    use: "Tutorial prompts, CTAs"
    aspect: "1:1"

  - id: thumbs_up
    pose: "giving enthusiastic thumbs up, big smile, cigar in other hand"
    use: "Achievement unlocked, success"
    aspect: "1:1"

  - id: worried
    pose: "concerned expression, hand on forehead, cigar lowered, sweating slightly"
    use: "Event damage, losses, warnings"
    aspect: "1:1"

  - id: riding_bill
    pose: "sitting on giant flying dollar bill like magic carpet, arms spread triumphantly"
    use: "Big win celebration, jackpot"
    aspect: "16:9"

  - id: thinking
    pose: "hand on chin thoughtfully, looking upward, contemplating"
    use: "Loading screens, decision moments"
    aspect: "1:1"

  - id: celebrating
    pose: "arms raised in victory, money raining around him, ecstatic"
    use: "Match victory, major achievement"
    aspect: "2:3"
```

### G3.4 Imperio Bill Currency Design

**Service**: Midjourney v6.1 or DALL-E 3
**Aspect Ratio**: --ar 16:9
**Style**: --s 100 (for detailed engraving)

```
CURRENCY PROMPT:
"Ornate $1000 currency bill design, vintage US dollar bill aesthetic,
central oval portrait frame with elderly businessman portrait placeholder
in fine line engraving crosshatch etching style, denomination '1000' in
decorative corners, 'IMPERIO INMOBILIARIO' as ornate header banner,
'ONE THOUSAND DOLLARS' text at bottom, decorative filigree borders
and scrollwork patterns, green money color scheme, serial number placeholders,
'NOT LEGAL TENDER' small text, highly detailed Victorian banknote
ornamental design, game currency prop art --ar 16:9 --s 100"
```

**Bill Variations**:
```yaml
bills:
  - denomination: 1000
    color: "green"
    use: "standard currency"

  - denomination: 100
    color: "green lighter"
    use: "small transactions"

  - denomination: 5000
    color: "gold tinted"
    use: "premium/special"
```

### G3.5 Logo Badge Design

**Service**: Ideogram 2.0 or Midjourney v6.1
**Aspect Ratio**: --ar 1:1
**Style**: Photorealistic embossed metal

```
LOGO BADGE PROMPT:
"Luxury game logo badge emblem, shield crest shape with 3D embossed depth,
golden crown at top with red jewel, main text 'IMPERIO' in large cream
colored serif font with gold outline and dark shadow, smaller text
'INMOBILIARIO' on red ribbon banner below, three gold stars beneath ribbon,
deep burgundy red gradient background (#8B0000 to #5C0000), ornate gold
border frame with bronze inner trim, metallic embossed medal appearance,
mobile game logo style, premium quality, transparent background
--ar 1:1 --s 100"
```

### G3.6 Dynasty Crests (17 Families)

**Service**: Midjourney v6.1
**Aspect Ratio**: --ar 1:1
**Style**: --s 150 --style raw

```
TEMPLATE PROMPT:
"Luxury heraldic family crest emblem for '{DYNASTY_NAME}' dynasty,
elegant shield shape with 3D embossed metallic finish,
{INDUSTRY_SYMBOL} as central golden motif, {ACCENT_COLOR} background
with gold and bronze ornamental border, crown or laurel wreath decoration,
premium mobile game dynasty badge style, aristocratic wealth aesthetic,
no text, transparent background --ar 1:1 --s 150 --style raw"
```

**Dynasty-Specific Elements**:

```yaml
dynasties:
  - name: Brecha / Brescia
    industry: "mining"
    symbol: "golden pickaxe crossed over mountain peak"
    accent: "deep amber and gold"

  - name: Romerazo / Romero
    industry: "finance and beer"
    symbol: "classical bank columns with wheat sheaf"
    accent: "navy blue and gold"

  - name: Gloriaz / Rodríguez
    industry: "fishing and food"
    symbol: "leaping fish with crown"
    accent: "ocean teal and silver"

  - name: Intercopia / Intercorp
    industry: "retail empire"
    symbol: "shopping bag with golden crown"
    accent: "royal purple and gold"

  - name: Hochi / Hochschild
    industry: "precious metals"
    symbol: "silver bars with condor"
    accent: "platinum silver and gray"

  - name: Benavidios / Benavides
    industry: "mining conglomerate"
    symbol: "mountain range with rising sun"
    accent: "bronze and earth tones"

  - name: Fishmanazo / Fishman
    industry: "fishing fleet"
    symbol: "golden anchor with fish"
    accent: "ocean blue and gold"

  - name: Acunazo / Acuña
    industry: "media empire"
    symbol: "TV screen with crown"
    accent: "crimson red and gold"

  - name: Dayer / Dyer
    industry: "textiles"
    symbol: "golden thread spool"
    accent: "royal purple and gold"

  - name: Mulderon / Mulder
    industry: "agriculture"
    symbol: "wheat sheaf with sun"
    accent: "harvest gold"

  - name: Mattita / Matta
    industry: "construction"
    symbol: "golden crane and building"
    accent: "steel gray and gold"

  - name: Ananos / Añaños
    industry: "beverages"
    symbol: "golden bottle with bubbles"
    accent: "deep blue and gold"

  - name: Ananos J / Añaños Jerí
    industry: "beverage expansion"
    symbol: "globe with bottle"
    accent: "emerald and gold"
```

### G3.7 Property Card Thumbnails

**Service**: Midjourney v6.1 or Flux Pro
**Aspect Ratio**: --ar 4:3
**Style**: --s 200 --style raw

```
TEMPLATE PROMPT:
"Beautiful {PROPERTY_TYPE} property in Lima Peru {DISTRICT} district,
{ARCHITECTURAL_STYLE}, golden hour lighting, front facade view,
semi-realistic architectural rendering, real estate photography style,
aspirational luxury property, clear blue sky, well-maintained,
mobile game property card thumbnail art --ar 4:3 --s 200 --style raw"
```

**Property Types**:

```yaml
property_thumbnails:
  - type: "luxury apartment"
    style: "modern glass balconies, coastal view"
    district: "Miraflores"

  - type: "corporate office tower"
    style: "sleek glass skyscraper, corporate plaza"
    district: "San Isidro"

  - type: "colonial mansion"
    style: "colorful restored colonial, ornate balcony"
    district: "Barranco"

  - type: "warehouse complex"
    style: "industrial metal buildings, loading docks"
    district: "Callao"

  - type: "shopping center"
    style: "modern mall entrance, palm trees"
    district: "Surco"

  - type: "hillside villa"
    style: "gated mansion, manicured gardens"
    district: "La Molina"

  - type: "historic building"
    style: "colonial architecture, Plaza de Armas area"
    district: "Centro"

  - type: "beach house"
    style: "modern coastal home, ocean view"
    district: "Chorrillos"
```

### G3.8 Achievement Badges

**Service**: Midjourney v6.1
**Aspect Ratio**: --ar 1:1
**Style**: --s 150 --style raw

```
TEMPLATE PROMPT:
"Luxury mobile game achievement badge, {ACHIEVEMENT_THEME},
circular medal design with 3D embossed depth, gold and bronze
metallic finish, {ACCENT_COLOR} enamel accent, ornate decorative
border, premium quality game reward icon, slight glow effect,
transparent background --ar 1:1 --s 150 --style raw"
```

**Achievement Badges**:

```yaml
badges:
  - id: A-TUT-01
    name: "Primer Casero"
    theme: "golden house key with sparkle"
    accent: "emerald green"

  - id: A-BET-01
    name: "Rompiste el Pacto"
    theme: "broken golden handshake"
    accent: "crimson red"

  - id: A-WIN-01
    name: "Magnate"
    theme: "golden crown over city skyline"
    accent: "royal purple"

  - id: A-WIN-05
    name: "Rey de Lima"
    theme: "elaborate golden throne"
    accent: "deep burgundy"

  - id: A-FAME-01
    name: "Famoso"
    theme: "golden star with camera flash"
    accent: "hot pink"

  - id: A-FAME-10
    name: "Leyenda"
    theme: "golden laurel wreath crown"
    accent: "imperial gold"

  - id: A-PROP-05
    name: "Portafolio"
    theme: "leather briefcase with buildings"
    accent: "rich brown"

  - id: A-PROP-10
    name: "Imperio"
    theme: "golden city map with empire pins"
    accent: "royal gold"

  - id: A-DEAL-01
    name: "Negociante"
    theme: "golden handshake with sparkle"
    accent: "teal"

  - id: A-DEAL-10
    name: "Tiburón"
    theme: "golden shark fin"
    accent: "navy blue"

  - id: A-SURV-01
    name: "Sobreviviente"
    theme: "golden phoenix rising from flames"
    accent: "orange fire"
```

### G3.9 Map Aerial View

**Service**: Midjourney v6.1 or Flux Pro
**Aspect Ratio**: --ar 1:1 (square for game board)
**Style**: --s 200 --style raw

```
MAP PROMPT:
"Aerial satellite view of Lima Peru metropolitan area, semi-realistic
digital painting style, Costa Verde coastline visible, Pacific Ocean
on left side, Andes foothills on right, major districts visible as
soft colored regions, golden hour lighting, slight tilt-shift effect,
game board map aesthetic, mobile strategy game quality, no text labels,
soft borders between districts --ar 1:1 --s 200 --style raw"
```

### G3.10 UI Frames and Buttons

**Service**: Midjourney v6.1
**Aspect Ratio**: varies (9-slice compatible)
**Style**: --s 100

```
UI FRAME PROMPT:
"Luxury mobile game UI frame, {FRAME_TYPE}, ornate gold and bronze
metallic border, {BACKGROUND_MATERIAL} center, 3D embossed depth,
premium tycoon game aesthetic, seamless tileable edges for 9-slice,
transparent or solid background --ar 1:1 --s 100"
```

**Frame Types**:

```yaml
ui_frames:
  - type: "property_card_frame"
    background: "cream parchment texture"
    border: "gold with bronze inner trim"

  - type: "button_normal"
    background: "teal gradient"
    border: "gold rounded corners"

  - type: "button_gold"
    background: "gold metallic gradient"
    border: "bronze embossed edge"

  - type: "panel_dark"
    background: "dark wood texture"
    border: "gold ornamental corners"

  - type: "modal_frame"
    background: "cream paper with subtle pattern"
    border: "elaborate gold filigree"
```

### G3.11 Event Illustrations (Tycoon Style)

**Service**: Midjourney v6.1
**Aspect Ratio**: --ar 4:3
**Style**: --s 200 --style raw

```
TEMPLATE PROMPT:
"Dramatic scene illustration for mobile tycoon game event: {EVENT_DESCRIPTION},
Lima Peru urban setting, cinematic composition, semi-realistic digital painting,
golden hour or dramatic lighting, {MOOD} atmosphere,
no identifiable faces, game event card art, premium mobile game quality
--ar 4:3 --s 200 --style raw"
```

**Key Event Prompts**:

```yaml
event_illustrations_tycoon:
  - id: E-045
    event: "Incendio"
    description: "luxury building engulfed in flames, firefighters arriving, dramatic orange glow"
    mood: "disaster, urgent"

  - id: E-051
    event: "Expropiación"
    description: "government officials with hard hats pointing at building, bulldozer nearby, legal documents"
    mood: "bureaucratic threat"

  - id: E-053
    event: "Anuncio de obras"
    description: "grand podium announcement, metro map on display, crowd of journalists"
    mood: "political spectacle"

  - id: E-020
    event: "Escándalo mediático"
    description: "paparazzi camera flashes, silhouette figure shielding face, newspaper headlines flying"
    mood: "scandal, exposure"

  - id: E-007
    event: "Protección forzada"
    description: "shadowy figures at building entrance at night, ominous lighting"
    mood: "threatening, noir"

  - id: G-01
    event: "Terremoto"
    description: "Lima skyline shaking, buildings swaying, dramatic cracks in ground"
    mood: "natural disaster"

  - id: G-02
    event: "Crisis económica"
    description: "stock market screens showing red, worried businesspeople, falling graphs"
    mood: "financial panic"
```

## G4: Batch Generation Workflows — Updated

### G4.1 Midjourney Batch Workflow

```
1. PREPARATION
   - Create prompt spreadsheet with all variations
   - Set up Discord server with Midjourney bot
   - Prepare naming convention: {ASSET_TYPE}_{ID}_{VERSION}

2. GENERATION
   - Use /imagine with template prompt
   - Generate 4 variations per prompt
   - Select best variation with V1-V4 buttons
   - Upscale selected with U1-U4

3. ORGANIZATION
   - Download all upscaled images
   - Rename following convention
   - Sort into asset type folders

4. POST-PROCESSING
   - Remove backgrounds (remove.bg or Photoshop)
   - Resize to target dimensions
   - Apply consistent color correction
   - Export final formats (PNG, SVG trace)
```

### G4.2 DALL-E 3 API Batch Script

```python
# scripts/generate_assets.py
import openai
from pathlib import Path
import json

client = openai.OpenAI()

def generate_asset(prompt: str, size: str = "1024x1024") -> str:
    """Generate single asset and return URL."""
    response = client.images.generate(
        model="dall-e-3",
        prompt=prompt,
        size=size,
        quality="hd",
        n=1
    )
    return response.data[0].url

def batch_generate(prompts_file: str, output_dir: str):
    """Batch generate assets from prompt file."""
    output_path = Path(output_dir)
    output_path.mkdir(parents=True, exist_ok=True)

    with open(prompts_file) as f:
        prompts = json.load(f)

    for item in prompts:
        print(f"Generating: {item['id']}")
        url = generate_asset(item['prompt'], item.get('size', '1024x1024'))

        # Download and save
        import requests
        response = requests.get(url)
        filename = f"{item['id']}.png"
        (output_path / filename).write_bytes(response.content)
        print(f"Saved: {filename}")

if __name__ == "__main__":
    batch_generate("prompts/stickers.json", "output/stickers")
```

### G4.3 Prompt File Format

```json
// prompts/mascot_poses.json
[
  {
    "id": "MAGNATE_STANDING",
    "prompt": "Distinguished elderly businessman character, 70 years old, white silver hair swept back elegantly, warm confident smile, holding thick Cuban cigar with visible smoke wisps, wearing perfectly tailored navy blue suit, white dress shirt, teal turquoise silk tie and pocket square, gold Rolex watch, caricature portrait style like premium mobile tycoon game mascot, NOT based on any real person, standing confidently with one hand on lapel, transparent background, semi-realistic digital painting --ar 2:3",
    "size": "1024x1536"
  },
  {
    "id": "MAGNATE_THUMBSUP",
    "prompt": "Distinguished elderly businessman character, 70 years old, white silver hair, warm smile, Cuban cigar in one hand, giving enthusiastic thumbs up with other hand, navy blue suit, teal tie, gold watch, mobile tycoon game mascot style, NOT based on real person, transparent background --ar 1:1",
    "size": "1024x1024"
  }
]
```

```json
// prompts/district_backdrops.json
[
  {
    "id": "BACKDROP_MIRAFLORES",
    "prompt": "Aerial view of Miraflores district Lima Peru, Costa Verde cliffs, Pacific Ocean, modern glass skyscrapers, Larcomar mall, golden hour lighting, semi-realistic digital painting, mobile tycoon game background, aspirational aesthetic, no text --ar 16:9",
    "size": "1792x1024"
  },
  {
    "id": "BACKDROP_SANISIDRO",
    "prompt": "Aerial view of San Isidro financial district Lima Peru, glass skyscrapers, El Olivar park olive trees, corporate towers, golden hour lighting, semi-realistic digital painting, mobile tycoon game background --ar 16:9",
    "size": "1792x1024"
  }
]
```

## G5: Post-Processing Pipeline

### G5.1 Background Removal

**Services**:
- remove.bg (API available, best quality)
- Photoshop "Remove Background"
- GIMP with alpha selection

```bash
# Using remove.bg CLI
removebg --api-key YOUR_KEY input.png output.png
```

### G5.2 SVG Tracing (for icons/crests)

**Tools**:
- Adobe Illustrator Image Trace
- Inkscape Trace Bitmap
- Vector Magic (online)

**Settings for clean game icons**:
```
Mode: Color
Colors: 6-8 max
Paths: Smooth
Corners: Fewer
Noise: Ignore small details
```

### G5.3 Batch Resize Script

```python
# scripts/resize_assets.py
from PIL import Image
from pathlib import Path

SIZES = {
    "skyline": (1920, 1080),
    "avatar": (256, 256),
    "icon": (128, 128),
    "sticker": (256, 256),
    "badge": (128, 128),
}

def resize_batch(input_dir: str, asset_type: str):
    target_size = SIZES[asset_type]
    input_path = Path(input_dir)
    output_path = input_path / "resized"
    output_path.mkdir(exist_ok=True)

    for img_file in input_path.glob("*.png"):
        img = Image.open(img_file)
        img = img.resize(target_size, Image.LANCZOS)
        img.save(output_path / img_file.name)
        print(f"Resized: {img_file.name}")

if __name__ == "__main__":
    resize_batch("assets/stickers", "sticker")
```

## G6: Quality Checklist

### G6.1 Per-Asset Verification

- [ ] Correct aspect ratio
- [ ] No text/watermarks
- [ ] Consistent style with other assets
- [ ] Colors match palette
- [ ] Background transparent (where needed)
- [ ] No identifiable real people
- [ ] Appropriate for all ages
- [ ] File size optimized (<500KB for game use)

### G6.2 Set Consistency Check

- [ ] All items in set have same style
- [ ] Color palette is consistent
- [ ] Line weights match
- [ ] Texture/grain is uniform
- [ ] Scale feels consistent

## G7: Cost Estimation

### G7.1 Full Asset Generation Budget

| Asset Category | Count | Cost/Item | Subtotal |
|----------------|-------|-----------|----------|
| Skyline Backplates | 12 | $0.50 | $6.00 |
| Press Avatars | 5 | $0.50 | $2.50 |
| Dynasty Crests | 17 | $0.50 | $8.50 |
| Stickers | 40 | $0.30 | $12.00 |
| Property Icons | 20 | $0.20 | $4.00 |
| Achievement Badges | 14 | $0.40 | $5.60 |
| Event Illustrations | 10 | $0.60 | $6.00 |
| Map Concepts | 3 | $0.50 | $1.50 |
| **TOTAL** | **121** | | **~$46** |

*Costs assume ~3 generations per final asset (iterations)*

### G7.2 Time Estimation

| Phase | Hours |
|-------|-------|
| Prompt writing & testing | 4-6 |
| Generation (batched) | 2-3 |
| Selection & curation | 2-3 |
| Post-processing | 4-6 |
| SVG tracing (icons) | 3-4 |
| Quality review | 1-2 |
| **TOTAL** | **16-24 hours** |

## G8: Legal & Safety Notes

### G8.1 AI-Generated Content Rights

| Service | Commercial Use | Ownership | Notes |
|---------|----------------|-----------|-------|
| Midjourney | ✅ (paid plans) | User owns | Pro/Mega plan required for commercial |
| DALL-E 3 | ✅ | User owns | OpenAI ToS applies |
| Leonardo.ai | ✅ (paid plans) | User owns | Check plan terms |
| Stable Diffusion | ✅ | User owns | Self-hosted = full control |

### G8.2 Content Safety Rules

1. **No Real People**: Never generate identifiable likenesses
2. **No Trademarks**: Avoid brand logos, even in background
3. **No Offensive Content**: Review all outputs before use
4. **Cultural Sensitivity**: Peruvian themes should be respectful
5. **Age Appropriate**: All content suitable for general audiences

### G8.3 Disclaimer for AI Assets

Include in game credits:
```
"Visual assets created with AI assistance (Midjourney, DALL-E).
All characters are fictional. Any resemblance to real persons is coincidental."
```

---

# VERIFICATION CHECKLIST

## v9 Content Preserved
- [x] All game modes (Quick/Standard/Long)
- [x] Room types and capacity (20 participants, 12 owners)
- [x] Teams/Dynasties (PARODY + REAL_LABEL sets)
- [x] Core loop phases
- [x] All economy formulas (Wealth, Rent, RiskScore, DevelopCost, OpsCost)
- [x] All actions and deals (6 templates)
- [x] Bank Loan wiring to InterestRates
- [x] Develop cost wiring to ConstructionEnforcement
- [x] Targeted event system with fairness caps
- [x] Infrastructure modifiers (INF-01, INF-02)
- [x] Expropriation wiring (E-051, E-051B, E-053)
- [x] Elections and world meters (6 meters)
- [x] Press system (5 personas)
- [x] Property deck (13 properties)
- [x] Market signals (10 signals)
- [x] Achievements list
- [x] Visual Spec Addendum (Appendix V)
- [x] Networking protocol (Appendix F)
- [x] TypeScript schemas (Appendix A1)

## v10 Improvements Integrated
- [x] Satellite Data strategy (external JSON files)
- [x] Visual Ledger phase (VISUAL_LEDGER, 4.0s blocking)
- [x] Deal Tray (non-blocking sidebar)
- [x] Busy Mode ("Modo Ocupado" / "Modo Concentrado")
- [x] JIT Round Seed with serverSecret
- [x] Delta packets (OP_DELTA)
- [x] Bot heuristics (Appendix A8)
- [x] Watcher betting on auctions

## v11 New Content
- [x] Open Source Foundation Matrix (Appendix O)
- [x] Colyseus integration examples (TypeScript)
- [x] Godot networking examples (GDScript)
- [x] Repository references with correct URLs
- [x] Viral mechanics (referrals, streaks)
- [x] Complete 52+ event deck with Spanish headlines
- [x] Phase timing diagram
- [x] Visual Ledger animation specs (GDScript)
- [x] AI Asset Generation Pipeline (Appendix G)
- [x] Service-specific prompts for all asset types
- [x] Batch generation workflows and scripts
- [x] Cost and time estimations
- [x] "Tycoon Luxury" visual style (Appendix V revised)
- [x] El Magnate mascot character specification
- [x] Imperio Bill currency design
- [x] Semi-realistic Lima cityscape direction

## Data File Schema Compatibility
- [x] `data/events.json` schema matches
- [x] `data/elections.json` schema matches
- [x] `data/signals.json` schema matches
- [x] `data/teams.json` schema matches
- [x] `data/locales_es.json` schema matches

---

# DOCUMENT METADATA

**Version**: 11.2
**Created**: 2026-01-29
**Updated**: 2026-01-29 (Visual style revised to "Tycoon Luxury")
**Lines**: ~4,500
**Sources**:
- imperio_inmobiliario_opus_spec_complete_v9_consolidated.md (1,120 lines)
- imperio_master_spec_v10.md (197 lines)
- data/*.json (existing satellite data)

**Supersedes**: v9 consolidated, v10 golden master

**Next Review**: After first playable prototype

---

*End of IMPERIO INMOBILIARIO v11 Unified Master Specification*
