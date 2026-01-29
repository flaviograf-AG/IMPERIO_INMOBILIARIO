# IMPERIO INMOBILIARIO — Master Logic Specification (v10 Golden Master)

## CHANGELOG (v9 → v10)
- **Architecture**: Adopted "Satellite Data" strategy. Content lists moved to external JSON files (`/data/`).
- **UX Fix**: Added **Visual Resolution Phase** (`VISUAL_LEDGER`) between INCOME and ACTION to visualize the "Invisible Accountant."
- **UI Fix**: Replaced blocking Deal Modals with a **Deal Tray** and added "Busy Mode" to prevent denial-of-service.
- **Security**: **Round Seed** is now generated Just-in-Time (salted with `serverSecret`) to prevent RNG prediction.
- **Network**: Added **Delta Packets** (`OP_DELTA`) to Appendix F to reduce bandwidth.
- **AI Logic**: Added **Appendix A8 Bot Heuristics** for Tutorial/Autopilot.

**Tagline:** Become **the richest** and **most famous** landlord dynasty in Lima.

**Language rule (hard):**
- All player-facing UI/strings are **Spanish** (loaded from `data/locales_es.json`).
- Logic variables and code comments are **English**.

---

## 0) Hard Constraints
- **Max Owners:** 12. Total Room Cap: 20.
- **Engine:** Godot 4.x (Web Export).
- **Server:** Node.js (Authoritative).
- **Input:** Mobile text inputs (Profile/Chat) must use **HTML DOM Overlays** to avoid Godot/WebGL virtual keyboard issues.

---

## 1) Game Modes
*(Config parameters loaded from `data/modes.json`)*
- **Quick:** 7 Rounds, 1 Global Shock.
- **Standard:** 10 Rounds, Elections R3/R7.
- **Long:** 14 Rounds, Elections R4/R10.

---

## 2) Roles & Virality
- **Watchers:**
  - **Rumor:** "Correr el rumor" (+0.2 RiskScore to target).
  - **Betting:** Can bet on Auction Winner (Watcher Fame reward).
- **Teams:** Defined in `data/teams.json`.

---

## 3) Onboarding
### 3.1 Tutorial Match
- **Trigger:** First login.
- **Setup:** 1 Human vs 3 Bots.
- **Script:** Relies on **Appendix A8** to ensure Bots underbid in R1 and fail to mitigate damage in R2.

---

## 4) Core Loop (State Machine)

### 4.1 Phases (Strict Order)
1.  **REVEAL**: Server broadcasts `roundSeed`, Market, Signals.
2.  **BID**: Simultaneous sealed bid.
3.  **INCOME_CALC**: Server calculates math (Instant).
4.  **VISUAL_LEDGER (New)**: **Blocking Phase (4.0s)**.
    - Client locks `ACTION` inputs.
    - UI animates the Income Receipt.
5.  **ACTION**: Player turns (Develop/Deal/Insure). Inputs Unlocked.
6.  **MOOD / VOTE**: Watcher influence or Election.
7.  **DEALS**: Resolve accepted deals.
8.  **EVENTS**: Resolve targeting.
9.  **PRESS**: Headlines.
10. **SNAPSHOT**: End of round state.

### 4.2 REVEAL & Seeding (Security Fix)
- **Vulnerability:** Sending `matchSeed` at start allows clients to predict all future coin flips.
- **Fix:** Server generates `serverSecret` (never sent).
- `roundSeed = hash(matchSeed, serverSecret, roundNumber)`.
- **Constraint:** Server sends `roundSeed` **ONLY** in the `REVEAL` packet for that specific round.

### 4.3 BID
- **Tie-break:** Amount > FameToken > CurrentFame > **Secure Hash**.
- **Hash:** `sha256(roundSeed + propertyId + ownerIds_sorted)`.

### 4.5 INCOME & VISUAL LEDGER (UX Fix)
**Server Logic (Instant):**
1.  Rent (+Signals, +Infra).
2.  Ops Cost (`floor((Props + Upgrades)/4)`).
3.  Debt Service.
4.  Leader Tax.
5.  **Forced Sales:** Triggered if Cash < 0.

**Client UX (Phase: VISUAL_LEDGER):**
- **Duration:** 4.0 seconds (Server enforced).
- **UI:** A "Receipt" animation plays center screen.
  - *Line 1:* "Rentas: +$8" (Green)
  - *Line 2:* "Costos: -$2" (Red)
  - *Total:* **+$6** (Animate cash counter).
- **Constraint:** Players cannot click "Action" buttons until the animation completes.

### 4.6 ACTION
Choose **one**: Develop, Deal, Insure, Security, PR, Audit, Romper Pacto, Pass.

---

## 5) Economy
- **Wealth:** `Cash + Σ(PropertyWorth) - Debt`
- **Rent:** `Base + Upgrade*2 + Bonuses`
- **RiskScore:** `1.0 + (0.35*Props) + (0.2*Fame) + (0.2*Wealth) + (0.25*Exposure) - (0.25*Compliance)`.

---

## 6) Actions & Deals (Anti-Spam Fix)

### 6.1 Standard Actions
- **Develop:** Cost wired to `ConstructionEnforcement` meter.
- **Insure:** Mitigation tables in `data/events.json`.

### 6.2 Deals (Deal Tray & Busy Toggle)
**Limits:**
- 1 Outgoing offer per owner per round.

**UI/UX Constraints:**
1.  **Non-Blocking:** Incoming deals appear in a **"Deal Tray"** (Sidebar/Toast), **NEVER** a blocking modal.
2.  **Busy Toggle:** Player can toggle **"Modo Ocupado"** (Busy Mode).
    - Effect: Server auto-rejects all incoming deals immediately.
    - Feedback to Sender: "El dueño está ocupado (Auto-reject)."
3.  **Bank Loan:** Repayment scheduled for *next* Income.

---

## 7) Targeted Events
- **Source:** `data/events.json`.
- **Fairness Caps:** Max 4 cash damage/round.
- **Dead-Wire Fix:** `E-053` activates Corridors; `E-051` targets Corridors.

---

## 8) Meters
- Political Stability, Security, etc. Effects defined in `data/elections.json`.

---

# Appendix A — Logic & Heuristics

### A8. Bot Heuristics (NEW)
**Role:** Fill empty seats in Tutorial/Public matches; Handle disconnects.

**H1. Economy Safety:**
- If `Cash < 2`, Action = **PASS**.
- Never Bid if `(Cash - Bid) < 2`.

**H2. Bidding Logic:**
- `Valuation = BasePrice + (Rent * 2)`.
- `Bid = min(Valuation, Cash - 1)`.
- *Tutorial Override:* Round 1 Bot always bids 1.

**H3. Deal Logic:**
- **Incoming:** Always REJECT (Reduces complexity).
- **Outgoing:** Never initiate.

**H4. Event Choices:**
- If Choice (Pay vs Penalty): Always PAY if `Cash >= Cost`.

---

# Appendix F — Network Protocol (Delta Update)

### F1. Strategy
1.  **SNAPSHOT:** Full state (sent on Join/Reconnect or Major Phase Change).
2.  **DELTA:** Small diffs (sent on high-frequency actions).

### F2. Packet Structure
**Delta (Bandwidth Optimization):**
```json
{
  "type": "OP_DELTA",
  "seq": 105,
  "changes": [
    { "op": "SET_CASH", "id": "O_01", "val": 15 },
    { "op": "SET_OWNER", "id": "P_05", "val": "O_02" }
  ]
}
---

### Part 2: The Satellite Data Files
*Save these in your project's `/data` folder.*

#### File: `data/properties.json`
```json
[
  { "id": "P-01", "nombre_es": "Depa Miraflores Vista", "district": "Miraflores", "tags": ["residencial", "costa", "premium_area"], "basePrice": 8, "baseRent": 3, "upgradeSlotsMax": 2, "buildQuality": "PREMIUM" },
  { "id": "P-02", "nombre_es": "Oficina San Isidro Prime", "district": "San Isidro", "tags": ["oficinas", "gobierno", "premium_area"], "basePrice": 9, "baseRent": 3, "upgradeSlotsMax": 2, "buildQuality": "PREMIUM" },
  { "id": "P-03", "nombre_es": "Loft Barranco", "district": "Barranco", "tags": ["cultura", "turismo", "costa"], "basePrice": 7, "baseRent": 3, "upgradeSlotsMax": 1, "buildQuality": "STANDARD" },
  { "id": "P-04", "nombre_es": "Local Jesus Maria", "district": "Jesus Maria/Lince", "tags": ["residencial", "gobierno"], "basePrice": 6, "baseRent": 2, "upgradeSlotsMax": 1, "buildQuality": "STANDARD" },
  { "id": "P-05", "nombre_es": "Almacen Callao", "district": "Callao", "tags": ["logistica", "zona_caliente"], "basePrice": 6, "baseRent": 2, "upgradeSlotsMax": 1, "buildQuality": "STANDARD" },
  { "id": "P-06", "nombre_es": "Casa Surco Familiar", "district": "Surco", "tags": ["residencial"], "basePrice": 7, "baseRent": 2, "upgradeSlotsMax": 2, "buildQuality": "STANDARD" },
  { "id": "P-07", "nombre_es": "Depa La Molina", "district": "La Molina", "tags": ["residencial", "premium_area"], "basePrice": 7, "baseRent": 2, "upgradeSlotsMax": 2, "buildQuality": "STANDARD" },
  { "id": "P-08", "nombre_es": "Tienda San Miguel", "district": "Magdalena/San Miguel", "tags": ["residencial", "costa"], "basePrice": 6, "baseRent": 2, "upgradeSlotsMax": 1, "buildQuality": "STANDARD" },
  { "id": "P-09", "nombre_es": "Casona Centro Historico", "district": "Centro", "tags": ["gobierno", "cultura"], "basePrice": 5, "baseRent": 2, "upgradeSlotsMax": 1, "buildQuality": "STANDARD" },
  { "id": "P-10", "nombre_es": "MiniDepa Chorrillos", "district": "Chorrillos", "tags": ["residencial", "costa"], "basePrice": 5, "baseRent": 2, "upgradeSlotsMax": 1, "buildQuality": "STANDARD" },
  { "id": "P-11", "nombre_es": "Depa Los Olivos", "district": "Independencia/Los Olivos", "tags": ["residencial"], "basePrice": 4, "baseRent": 2, "upgradeSlotsMax": 1, "buildQuality": "STANDARD" },
  { "id": "P-12", "nombre_es": "Local Ate", "district": "Ate/SJL", "tags": ["logistica", "residencial"], "basePrice": 4, "baseRent": 2, "upgradeSlotsMax": 1, "buildQuality": "STANDARD" },
  { "id": "P-20", "nombre_es": "Torre Financiera", "district": "San Isidro", "tags": ["oficinas", "gobierno", "premium_area"], "basePrice": 10, "baseRent": 4, "upgradeSlotsMax": 2, "buildQuality": "PREMIUM" }
]