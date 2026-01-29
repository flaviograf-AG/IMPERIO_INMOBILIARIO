# IMPERIO INMOBILIARIO — Development Specification (Claude Opus Ready, v9 — consolidated)
## CHANGELOG (v8 → v9 consolidated)
- Added **Visual Spec Addendum** (Appendix V): art direction, UI kit, motion/SFX, MapView spec, asset list, and image-generator prompts.
- Fixed “dead wires”:
  - **Infrastructure corridor activation** so **Expropiacion** is reachable: added **E-053 “Anuncio de obras”** and E-051 fallback.
  - **InterestRates** now affects **Bank Loan** repayment.
  - **Construction Enforcement** now affects **Develop** cost.
- Added **Appendix A7 Property Deck** (minimal set) to make the spec self-contained and strictly simulatable.

**Tagline:** Become **the richest** and **most famous** landlord dynasty in Lima—while the city, the press, and Public Mood try to take you down.

**Design goals**
- **Fast to learn (≤ 60 seconds):** each round, each Owner does **one Bid** + **one Action**.
- **High virality:** public rooms, Watchers influence outcomes, shareable “Portada” + mid-match “Logros”.
- **Low infra + low hardware:** turn-based, 2D UI-first, Godot Web export, authoritative server.
- **Tycoon feel without complexity:** demand cycles + portfolio overhead + upgrades.
- **Strong Lima/Peru flavor:** Spanish-only player content, last-10-years macro themes, safe Peruvian humor.

**Language rule (hard):**
- All **player-facing UI, cards, headlines, and prompts are Spanish**.
- This document is in English; every content pack includes Spanish strings.

---

## 0) Hard constraints (must not change)
- Max **Owners = 12** in Standard mode.
- Up to **20 participants** may join a room (Owners + Watchers).
- Unlimited Watchers (within room cap).
- Events are mostly **targeted at a single Owner**, with probability increasing with:
  - number of properties owned
  - Wealth and Fame (media attention)
- Insurance has cost and benefit (risk reduction + payout).
- Catch-up mechanics are mandatory.
- Deals are template-only (no free-form).
- Graf Design System is **admin UI only**.
- Public rooms are viewable; profile share pages are **unlisted by default**.

---

## 1) Game modes (length + owner cap)
All modes share rules; only parameters differ.

### 1.1 Quick (viral default)
- Owners: 4–12
- Rounds: 7
- Timers: 25s bid / 25s action
- Market Row: 5 properties
- Targeted event slots per round: 1 (+1 if Owners ≥ 9)
- Global shock cap: 1 per match

### 1.2 Standard
- Owners: 4–12
- Rounds: 10
- Timers: 35s bid / 35s action
- Market Row: 6 properties
- Targeted slots: 1 (+1 if Owners ≥ 7) (+1 if Owners ≥ 11)
- Elections: rounds 3 and 7 (VOTE phase replaces Mood on those rounds)
- Global shock cap: 2 per match

### 1.3 Long (strategic)
- Owners: 4–8
- Rounds: 14
- Timers: 40s bid / 40s action
- Market Row: 6 properties
- Targeted slots: 2 (+1 if Owners ≥ 7)
- Elections: rounds 4 and 10 (VOTE phase replaces Mood on those rounds)
- Global shock cap: 2 per match

Room host chooses mode at creation.

---

## 2) Roles, room capacity, identity, and virality loops

### 2.1 Room capacity (up to 20 participants)
- Up to **20 participants** may join a room.
- Only **12** can be Owners at once.
- Remaining participants are Watchers.

### 2.2 Owners
- One bid per round.
- One action per round.
- One deal offer per round.

### 2.3 Watchers
- Open join in public rooms.
- One tap per round for Mood; occasional award votes.
- Optional reaction emojis (no chat required).

**Watcher “Rumor” button (one per match):**
- Once per match, a Watcher may nominate an Owner to receive `+0.2 RiskScore` next round.
- Rate limit: 1 Rumor per Watcher per match.
- Server caps total Rumor influence per Owner per round to +0.6.
- UI label (Spanish): “Correr el rumor”.

### 2.4 Room types
- **Public**: open join, listed in lobby.
- **Friends**: token/QR join.
- Optional: lock seats after round 1.

### 2.5 Teams (Dynasties) — two sets (public safe default)
Teams are fantasy labels for prestige rivalry.

#### 2.5.1 Team sets
- **Public rooms (default): PARODY set** (safer, satirical, non-identifying).
- **Friends rooms (default): REAL-LABEL set** (team labels only), with mandatory disclaimer.

Host may override:
- Public rooms may choose REAL-LABEL only if host enables “Real labels (private-like)” and accepts disclaimer.
- Friends rooms may choose PARODY for streaming, etc.

#### 2.5.2 Mandatory disclaimer (Spanish, shown in lobby + share pages)
> “Los nombres de dinastias se usan solo como fantasia/satira. No hay afiliacion ni representacion de personas u organizaciones reales.”

#### 2.5.3 REAL-LABEL set (17)
Brescia, Romero, Rodríguez (Gloria), Rodríguez Pastor (Intercorp), Hochschild, Benavides, Arias Vargas, Navarro Grau, Marsano, Lindley, Fishman, Acuña, Dyer, Mulder, Matta, Añaños, Añaños Jerí.

#### 2.5.4 PARODY set (17)
Brecha, Romerazo, Gloriaz, Intercopia, Hochi, Benavidios, Arias V, Navarro G, Marsanito, Lindlitas, Fishmanazo, Acunazo, Dayer, Mulderon, Mattita, Ananos, Ananos J.

#### 2.5.5 Team selection
Players may:
- Pick a team during profile setup, or
- Auto-assign randomly on first join (change once pre-match).
Room option:
- `Unique Teams` (default OFF): if ON, each team can be used by only one Owner.

### 2.6 Viral/retention hooks (proven patterns, adapted)
- **Portada** share card at end of match (1200×630 + 1080×1920).
- Mid-match **Logros** trophy cards (shareable).
- **Dynasty Ladder** (weekly seasons).
- “**Revancha al toque**” rematch flow.
- Watcher awards: “**Favorito del publico**”, “**Ampay del dia**”.

---

## 3) Onboarding (time-to-fun)

### 3.1 Mandatory first-time Tutorial Match
Triggered: first time an account joins any room.

Format:
- 2 rounds
- 3 Owners minimum (fill missing with Bots)
- Forced outcomes:
  - Round 1: user wins a cheap auction with suggested bid
  - Round 2: user is targeted by a mild event and sees insurance mitigate it

Constraints:
- Coach overlays: max 2 Spanish sentences per phase.
- Reward: achievement `A-TUT-01 "Primer Casero"` + cosmetic badge.
- User can skip after completing once.

---

## 4) Core loop (per round) + tempo markers

### 4.1 Phases (authoritative; deterministic order)
Normal rounds:
- REVEAL → BID → INCOME → ACTION → MOOD → DEALS → EVENTS → PRESS → (optional AWARDS) → next round

Election rounds (replace Mood with Vote):
- REVEAL → BID → INCOME → ACTION → **VOTE** → DEALS → EVENTS → PRESS → (optional AWARDS) → next round

**Hard ordering rule (no ambiguity):**
1) Resolve bids  
2) Apply Income (rent + signals + ops + debt + scheduled + leader tax)  
3) Collect actions  
4) Resolve Mood/Vote result (queued effects)  
5) Resolve deals at DEALS phase (post-action ownership becomes authoritative)  
6) Select targets and resolve events (apply caps)  
7) Generate headlines  
8) Emit achievements  
9) Emit snapshots

Timeouts:
- no bid → no bid
- no action → Pass
- no mood/vote → no vote
Server proceeds regardless.

### 4.2 REVEAL
Server reveals:
- Market Row (properties)
- Two Market Signals (tycoon layer)
- Two Mood options (non-election rounds)
- Special Round rule if applicable

### 4.3 BID (sealed, simultaneous)
Each Owner may bid on one property:
- (propertyId, amount)
- optional: spend 1 Fame as tie-break token (max 1 per round)

**Payment model (winner-only; no escrow):**
- Only the winning Owner pays the bid amount.
- Losing Owners do not pay anything (no refunds exist).

**Tie-break order:**
1) higher bid amount  
2) Fame token used (yes > no)  
3) higher current Fame  
4) **Deterministic seeded coin flip** (see 4.4)

### 4.4 Deterministic seeded coin flip (hash)
Used for any “coin flip” tie-break in the game.
```
winnerIndex = hash(roundSeed, propertyId, join(sorted(tiedOwnerIds), ",")) % len(tiedOwnerIds)
winnerOwnerId = sorted(tiedOwnerIds)[winnerIndex]
```
- `roundSeed` is generated at MATCH_INIT and rotated each round with `hash(matchSeed, roundNumber)`.
- Hash must be stable across server restarts (e.g., SHA-256).

### 4.5 INCOME
Apply:
- Rent income (with Market Signals)
- Operating costs (portfolio overhead)
- Debt service
- Leader friction tax (“Impuesto a la fama”)
- Scheduled effects
- Forced sales if cash < 0

### 4.6 ACTION
Choose exactly one:
- Develop
- Deal (offer)
- Insure
- Security Contract
- PR Push / Compliance Audit
- **Romper Pacto** (once per match)
- Pass

### 4.7 MOOD (Watchers; non-election rounds)
Watchers vote between 2 options (one tap). Winning modifier is queued to apply next round.

### 4.8 VOTE (election rounds; replaces Mood)
Owners choose:
- Candidate A or B
- Contribution 0/1/2 cash (counts as “support points”)
Watchers vote Candidate A or B (one tap).

**Election winner determination (low-input, clear):**
- ScoreA = Σ(ownerContributionToA)  
- ScoreB = Σ(ownerContributionToB)  
- If ScoreA != ScoreB → winner is higher score  
- If tie → **Watcher vote decides** (this is the “Watcher Mood” tie-break for elections)  
- If watcher vote tie → deterministic coin flip (4.4)

Effects:
- Winner bundle applies for **next 2 rounds** (meters + weight shifts).
- Any Owner contributing >0 increases Political Exposure by +1 (max 3).

### 4.9 DEALS
Resolve accepted deals deterministically (Appendix A2).

### 4.10 EVENTS
Resolve targeted events and rare global events, applying fairness caps.

### 4.11 PRESS
Generate 1–3 Spanish headlines in the match Press Persona style.

### 4.12 AWARDS (optional)
Server nominates owners for:
- Favorito del publico
- Ampay del dia
Watchers vote (1 tap). Awards applied at end of match.

### 4.13 Special rounds (tempo markers; mandatory)
To ensure “moment density”:
- Quick: R3 Guerra de Distrito, R6 Ventana Bancaria
- Standard: R3 Guerra de Distrito, R5 Ventana Bancaria, R9 Subasta de Prestigio
- Long: R4 Guerra de Distrito, R8 Ventana Bancaria, R13 Subasta de Prestigio

Effects:
- Guerra de Distrito: Market Row biased to one district; completing a set this round grants +1 Fame.
- Ventana Bancaria: Loan improves (+4 now, repay −5), Media/Scandal weight +10%.
- Subasta de Prestigio: auction win grants +2 Fame; Watchers see “Portada preview”.

---

## 5) Economy, tycoon layer, and scoring

### 5.1 Resources
Cash, Debt, Fame, Compliance (0–3), Political Exposure (0–3).

### 5.2 Wealth (primary win)
```
Wealth = Cash + Σ(PropertyWorth) − Debt
PropertyWorth = BasePrice + (UpgradeLevel * 2) + WorthBonuses
```

### 5.3 Rent
```
Rent = BaseRent + (UpgradeLevel * 2) + DistrictSetBonus + MarketSignalBonus + TemporaryRentBonus
```

### 5.4 District set bonus
2+ properties in same district: +1 rent per property in that district.

### 5.5 Starting values
Standard baseline: Cash 10 (Quick 9, Long 11), Debt 0, Fame 0, Compliance 1, Political Exposure 0, Debt cap 10.

### 5.6 Market Signals (business-sim demand cycles)
Each round reveal shows 2 signals that modify rent by tag (±1; per property cap ±1).
Signals list is deterministic (Appendix A4).

### 5.7 Operating costs (portfolio overhead; leader friction)
Per round:
```
OpsCost = floor( (NumProperties + TotalUpgrades) / 4 )
```

### 5.8 Leader friction (Income-phase)
**Impuesto a la fama:**
- top Wealth quartile: pay +1 cash
- #1 Wealth pays +2 cash if Wealth > 1.2 * Wealth(#2)

### 5.9 Error copy (Spanish; safe humor)
If an action requires cash you do not have:
- “**No te alcanza, causa.**”

---

## 6) Actions and deals (template-only; alliances; betrayal moment)

### 6.1 Develop (wired to Construction Enforcement)
**Base cost:** 3 cash  
**Meter wiring:**
```
DevelopCost = 3 + floor((ConstructionEnforcement + 1) / 2)
```
So:
- Enforcement 0 → cost 3
- Enforcement 1 → cost 4
- Enforcement 2 → cost 4
- Enforcement 3 → cost 5

Effect: +2 rent permanently. Adds +0.05 RiskScore next round.
### 6.2 Deals (templates; server-enforced)
One outgoing offer per owner per round. One counter max. Offers expire at end of DEALS phase.

Templates:
1) Cash for Property  
2) Property Swap  
3) Bank Loan (principal +3 now; repay next Income = Principal + 1 + InterestRates at time of loan; during Ventana Bancaria principal +4; if cannot repay → amount converts to Debt and Fame −1 “Deudor moroso”)  4) Election Pact (election rounds): “Si apoyas al candidato X, te pago 1 en el proximo ingreso.”  
5) Non-compete (district, 1 round): “No pujo en Distrito D el proximo round; me pagas 1 ahora.”  
6) Security Escort: “Te pago 1; quedas Protegido hasta fin del proximo round.”

#### Bank Loan repayment (authoritative; wired to InterestRates)
Bank Loan schedules repayment in the **next INCOME** phase.

- **Principal:** +3 cash now (or +4 during Ventana Bancaria)
- **Repayment:**
```
Repay = Principal + 1 + InterestRates
```
- InterestRates is the meter value (0–3) **at the moment the loan is taken**.
- Record `scheduledRepayAmount` and `scheduledRepayRound` on the deal so later meter changes do not retroactively change repayment.

If the Owner cannot pay the scheduled repayment during INCOME:
- The unpaid amount converts to **Debt** (same amount).
- Fame −1 with Spanish headline flavor (“Deudor moroso”).


### 6.3 Betrayal mechanic: Romper Pacto (once per match)
Eligibility: Owner currently has an active Non-compete pact.

Effect:
- Cancel the pact immediately.
- Fame +1 now.
- Next round RiskScore +1.0.
- Triggers trophy-card `A-BET-01 "Rompiste el Pacto"`.

### 6.4 Insurance (property-level)
None→Basic cost 1; Basic→Full cost 2; None→Full cost 3.
Insurance mitigation is **fully defined** in Appendix B4 (Mitigation Tables).

### 6.5 Security Contract (owner-level)
Cost 1. Duration until end of next round.
Mitigation: extortion severity −1 and blocks Fame loss from extortion cards.

### 6.6 PR / Compliance
- PR Push: Fame +1 now; next round RiskScore +0.5
- Compliance Audit: Compliance +1 (max 3); next round RiskScore −0.5

### 6.7 Fame spend (once per round)
- Yape de emergencia: spend 2 Fame → +1 cash (cap once per match)
- Control de danos: spend 2 Fame → reduce event cash penalty by 1
- Cortina de humo: spend 3 Fame → cancel one Fame loss

### 6.8 Forced sales
If cash < 0 after Income: sell lowest-basePrice at 80% basePrice until cash ≥ 0 or none.

---

## 7) Targeted event system + fairness constraints

### 7.1 RiskScore (explicit formula)
```
RiskScore =
  1.0
  + 0.35 * NumProperties
  + 0.20 * FameTier
  + 0.20 * WealthTier
  + 0.25 * PoliticalExposure
  - 0.25 * Compliance
  + DistrictRiskBonus
  + AttentionBonus
  - ProtectionBonus
  - RecentTargetShield
  + RumorBonus
```

- DistrictRiskBonus: +0.2 if owns logistica-tag property; +0.1 if owns Centro/gobierno-tag; +0.1 if many costa-tag
- AttentionBonus: +0.2 if PR Push last round
- ProtectionBonus: −0.2 if Security Contract active
- RecentTargetShield: −0.6 if targeted last round
- RumorBonus: sum of watcher rumors, capped +0.6
Clamp [0.5, 3.5].

### 7.2 Category weights
Seguridad 25%, Licencias 25%, Prensa 25%, Incidentes 20%, Global 5%.
Special rounds can shift weights (Ventana Bancaria: Prensa +10%).

### 7.3 Fairness caps (must enforce)
- Round cash damage cap per Owner from events: max 4 cash; overflow becomes rent penalty.
- Expropriation: max 1 per owner per match.
- Global shocks: mode cap and never more than 1 per 3 rounds.
- No double-target per round.
- Mercy rule: 0 properties → cannot be hit by property-loss events.
- Hard-hit spacing: within any 2-round window, an owner may receive at most 1 event whose pre-mitigation cash loss ≥3.
- Scandal bias: at least 60% of Prensa events are Fame-heavy (cash impact ≤1).

### 7.4 AutoChoice (keep gameplay fast)
Some event cards define “choice” outcomes (pay vs penalty). These are resolved automatically by the server:

- If a choice includes `payCash = X` to avoid `penaltyValue = Y`, and `cash >= X`:
  - pay if `X <= Y`
  - otherwise take penalty
- If cash < X: take penalty
- If both options are non-numeric: choose the option that reduces next-round RiskScore.

Optional pre-match preference:
- “Siempre pagar para evitar penalidades: Si/No”
If “No”, AutoChoice only pays if `X < Y`.


### 7.5 Infrastructure modifiers and Expropriation wiring (dead-wire fix)
Infrastructure modifiers (**INF-01/INF-02**) are a separate system:
- Each INF attaches to a district and lasts **2 rounds**.
- While INF active on a district: `enablesExpropriation = true` for that district.
- INF should be shown in MapView as an animated corridor overlay (“METRO” / “VIA”).

**E-051 Expropiacion resolution rule (no “no-op”):**
- If at least one district has `enablesExpropriation=true`, E-051 must target an Owner who owns a property in an enabled district (with one reroll of target owner if needed).
- If there are no enabled districts anywhere (should be rare once E-053 exists), resolve fallback **E-051B “Aviso de derecho de via”**:
  - cashDelta −1
  - apply `tradeLockedUntilRound = currentRound + 2` on a random owned property
  - headline: blueprint/“trazo” theme

**New event (add to deck): E-053 “Anuncio de obras”**
- Activates either INF-01 or INF-02 on a chosen district for 2 rounds.
- District selection bias:
  - 50% choose among districts in current Market Row
  - 50% choose among districts with the most owned properties
- Immediate district effects while active:
  - rent +1 for properties in district
  - worth +2 for properties in district

---

## 8) Elections and world meters
Meters: Political Stability, BCRP Independence, Security, Inflation, Interest Rates, Construction Enforcement (0–3).
Election bundles apply for next 2 rounds (Appendix A5).

---

## 9) Press system (Spanish-only; safe Peruvian humor)
Press personas: Diario serio, Portada chicha, Programa de la tarde, Radio con llamadas, Boletin municipal.

Headlines are template-driven Spanish strings. Mild colloquialisms allowed (causa, al toque, asu, palta, con fe, chambea, yape).
No slurs, no harassment, no targeting protected characteristics.

Real-world inspiration uses **partial-name flavor tokens** only; never framed as new factual claims.

---

## 10) Technology stack (open source; low infra)

### 10.1 Client
- Godot 4.x (Web export primary).
- Render: **Compatibility** renderer for widest device support (WebGL2).
- Avoid threads; avoid heavy shaders; keep 2D UI-first.
- Hybrid template approach: start from a Godot card/board UI template and reskin.

**UI split:**
- Admin/menus use Graf Design System tokens + Material Symbols.
- Match uses “Imperio” aesthetics (newspaper, skyline, district map overlays).

### 10.2 Server
- Node.js + TypeScript
- WebSockets (authoritative state machine)
- PostgreSQL persistence
- Redis optional for rate limiting

### 10.3 Infra
- One VPS + Nginx TLS proxy
- Share assets: local store + TTL cleanup (MVP), migrate to S3 later

---

## 11) Anti-abuse
- Phase gating, strict validation.
- Session + IP rate limiting.
- Idempotency keys + monotonic nonce.
- Reconnect 60s (after that autopilot continues; seat may be locked by host rules).

---

# Appendix A — Deterministic content packs (single source of truth)

All player-facing strings are Spanish and stored in these JSON packs.

## Appendix A1 — Data schemas (authoritative types)

### A1.1 Core types (TypeScript)
```ts
type Id = string;

type RoomType = "PUBLIC" | "FRIENDS";
type Role = "OWNER" | "WATCHER";
type Phase = "REVEAL" | "BID" | "INCOME" | "ACTION" | "MOOD" | "VOTE" | "DEALS" | "EVENTS" | "PRESS" | "AWARDS" | "END";

type InsuranceLevel = "NONE" | "BASIC" | "FULL";

type OwnerState = {
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
  compliance: 0|1|2|3;
  politicalExposure: 0|1|2|3;

  protectedUntilRound?: number;
  brokePactUsed: boolean;
  yapeUsed: boolean;
  lastTargetedRound?: number;

  preferAutoPay: boolean;

  properties: Id[];
  achievements: string[];
};

type PropertyState = {
  propertyId: Id;
  district: string;
  tags: string[]; // must be subset of Tag enums
  basePrice: number;
  baseRent: number;
  upgradeSlotsMax: 0|1|2;
  upgradeLevel: 0|1|2;
  insurance: InsuranceLevel;
  buildQuality: "STANDARD" | "PREMIUM";
  ownerId?: Id;
  tempRentBonus?: { delta: number; untilRound: number }[];
  tempWorthBonus?: { delta: number; untilRound: number }[];
  tradeLockedUntilRound?: number;
};

type DealTemplateId =
  "CASH_FOR_PROPERTY"|"SWAP"|"BANK_LOAN"|"ELECTION_PACT"|"NON_COMPETE"|"SECURITY_ESCORT";

type DealState = {
  dealId: Id;
  templateId: DealTemplateId;
  fromOwnerId: Id;
  toOwnerId: Id;
  createdRound: number;
  expiresAtPhase: "DEALS";
  status: "PENDING"|"COUNTERED"|"ACCEPTED"|"REJECTED"|"EXPIRED"|"RESOLVED"|"INVALID";
    fields: Record<string, any>;
  counterFields?: Record<string, any>;

  // BANK_LOAN only (authoritative repayment scheduling)
  scheduledRepayAmount?: number;
  scheduledRepayRound?: number;
};


type FameSpendUsed = "YAPE"|"CONTROL_DANOS"|"CORTINA";

type EventResult = {
  cardId: Id;
  targetOwnerId?: Id;
  targetPropertyId?: Id;
  effectsApplied: {
    cashDelta: number;
    debtDelta: number;
    fameDelta: number;
    rentPenaltyApplied?: { propertyId: Id; delta: number; duration: number }[];
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
};

type MatchStateSnapshot = {
  roomId: Id;
  mode: "QUICK"|"STANDARD"|"LONG";
  round: number;
  phase: Phase;
  phaseEndsAtTs: number;

  owners: OwnerState[];
  properties: PropertyState[];
  deals: DealState[];

  marketRow: Id[];
  marketSignals: { id: Id; nombre_es: string; tagMods: Record<string, number> }[];
  moodOptions?: { id: Id; nombre_es: string; effect: any }[];
  electionOptions?: { id: Id; nombre_es: string; bundle: any }[];

  meters: Record<string, number>;

  infra?: { infraId: string; district: string; untilRound: number; rentDelta: number; worthDelta: number; enablesExpropriation: boolean }[];

  pressPersonaId: Id;
  pressTicker: string[];
  lastEventResults?: EventResult[];

  watcherCount: number;
};
```

### A1.2 Tag enums (closed set)
Economic tags (Market Signals):
- turismo, oficinas, logistica, residencial, gobierno, cultura, costa

Flavor tags (risk/aesthetic only):
- premium_area, corredor_obra, zona_caliente

---

## Appendix A2 — Deal engine rules (deterministic)

**Lifecycle**
- Each Owner can have at most 1 outgoing offer per round.
- Offer expires at end of DEALS phase of same round.
- One counter-offer maximum.

**Resolution order**
- All deal acceptances/counters collected during ACTION phase.
- At DEALS phase:
  1) Expire stale offers.
  2) Resolve ACCEPTED deals deterministically (sorted by dealId).
  3) If a referenced property is not owned by sender at DEALS phase → deal becomes INVALID.

**Conflict rules**
- A property may be transferred at most once per round via deals.
- If two accepted deals attempt to transfer the same property:
  - the lowest dealId resolves
  - others become INVALID

**Forced sales**
- Occur during INCOME and can invalidate later deals.

---

## Appendix A3 — State machine (fully enumerated)
LOBBY → MATCH_INIT → for each round:
- REVEAL → BID → INCOME → ACTION → (MOOD or VOTE) → DEALS → EVENTS → PRESS → (optional AWARDS)
→ next round → END

---

## Appendix A4 — Market Signals list (deterministic; Spanish)
- S-01 "Boom Airbnb" → {turismo:+1,costa:+1}
- S-02 "Oficinas flojas" → {oficinas:-1}
- S-03 "Puerto full" → {logistica:+1}
- S-04 "Lima de noche" → {cultura:+1}
- S-05 "Familias compran" → {residencial:+1}
- S-06 "Burocracia al 200%" → {gobierno:-1}
- S-07 "Turismo asustado" → {turismo:-1,costa:-1}
- S-08 "Rebote post-crisis" → {residencial:+1,oficinas:+1}
- S-09 "Flete caro" → {logistica:-1}
- S-10 "Moda Barranco" → {cultura:+1,turismo:+1}

No repeat of the same signal in consecutive rounds.

---

## Appendix A5 — Elections (deterministic; Spanish)
Candidates are revealed as A/B on election rounds. Bundles apply next 2 rounds.

Examples:
- C-01 "General Orden" — "Mano dura, pero con licencia"
  - meters: Security +1, ConstructionEnforcement +1
  - weightMods: Extortion -10%, Licencias +10%
- C-02 "Tecnocrata Pro" — "Estabilidad, por favor"
  - meters: BCRP +1, InterestRates -1, PoliticalStability +1
- C-03 "Shock Reformista" — "Corta la grasa"
  - meters: PoliticalStability +1, InterestRates +1, Inflation -1
  - weightMods: Prensa +10%
- C-04 "Estado Expansivo" — "Mas Estado, mas obra"
  - meters: Inflation +1, BCRP -1, PoliticalStability -1
  - weightMods: Licencias +15%

Tie-break: Watcher vote, then seeded coin flip.

---

## Appendix A6 — Achievements (Spanish)
Include at minimum:
- A-TUT-01 "Primer Casero"
- A-BET-01 "Rompiste el Pacto"
- (others as in v4 set)

---


## Appendix A7 — Property Deck (minimal self-contained set; deterministic)
**Rule:** Property deck is a finite list. Market Row draws without replacement; when deck ends, reshuffle discard into deck with deterministic shuffle using `matchSeed`.

```yaml
properties:
  - id: P-01
    nombre_es: "Depa Miraflores Vista"
    district: "Miraflores"
    tags: [residencial, costa, premium_area]
    basePrice: 8
    baseRent: 3
    upgradeSlotsMax: 2
    buildQuality: "PREMIUM"
  - id: P-02
    nombre_es: "Oficina San Isidro Prime"
    district: "San Isidro"
    tags: [oficinas, gobierno, premium_area]
    basePrice: 9
    baseRent: 3
    upgradeSlotsMax: 2
    buildQuality: "PREMIUM"
  - id: P-03
    nombre_es: "Loft Barranco"
    district: "Barranco"
    tags: [cultura, turismo, costa]
    basePrice: 7
    baseRent: 3
    upgradeSlotsMax: 1
    buildQuality: "STANDARD"
  - id: P-04
    nombre_es: "Local Jesus Maria"
    district: "Jesus Maria/Lince"
    tags: [residencial, gobierno]
    basePrice: 6
    baseRent: 2
    upgradeSlotsMax: 1
    buildQuality: "STANDARD"
  - id: P-05
    nombre_es: "Almacen Callao"
    district: "Callao"
    tags: [logistica, zona_caliente]
    basePrice: 6
    baseRent: 2
    upgradeSlotsMax: 1
    buildQuality: "STANDARD"
  - id: P-06
    nombre_es: "Casa Surco Familiar"
    district: "Surco"
    tags: [residencial]
    basePrice: 7
    baseRent: 2
    upgradeSlotsMax: 2
    buildQuality: "STANDARD"
  - id: P-07
    nombre_es: "Depa La Molina"
    district: "La Molina"
    tags: [residencial, premium_area]
    basePrice: 7
    baseRent: 2
    upgradeSlotsMax: 2
    buildQuality: "STANDARD"
  - id: P-08
    nombre_es: "Tienda San Miguel"
    district: "Magdalena/San Miguel"
    tags: [residencial, costa]
    basePrice: 6
    baseRent: 2
    upgradeSlotsMax: 1
    buildQuality: "STANDARD"
  - id: P-09
    nombre_es: "Casona Centro Historico"
    district: "Centro"
    tags: [gobierno, cultura]
    basePrice: 5
    baseRent: 2
    upgradeSlotsMax: 1
    buildQuality: "STANDARD"
  - id: P-10
    nombre_es: "MiniDepa Chorrillos"
    district: "Chorrillos"
    tags: [residencial, costa]
    basePrice: 5
    baseRent: 2
    upgradeSlotsMax: 1
    buildQuality: "STANDARD"
  - id: P-11
    nombre_es: "Depa Los Olivos"
    district: "Independencia/Los Olivos"
    tags: [residencial]
    basePrice: 4
    baseRent: 2
    upgradeSlotsMax: 1
    buildQuality: "STANDARD"
  - id: P-12
    nombre_es: "Local Ate"
    district: "Ate/SJL"
    tags: [logistica, residencial]
    basePrice: 4
    baseRent: 2
    upgradeSlotsMax: 1
    buildQuality: "STANDARD"
  - id: P-20
    nombre_es: "Torre Financiera"
    district: "San Isidro"
    tags: [oficinas, gobierno, premium_area]
    basePrice: 10
    baseRent: 4
    upgradeSlotsMax: 2
    buildQuality: "PREMIUM"
```


# Appendix B — Event deck + mitigation (authoritative)


## Appendix B0 — Event deck patch (dead-wire fix; authoritative)
Add this event to the deck:
- **E-053 “Anuncio de obras”** → activates **INF-01** or **INF-02** on a district for 2 rounds (see Section 7.5).

Modify **E-051 Expropiacion**:
- If no INF-enabled districts exist anywhere, resolve fallback **E-051B “Aviso de derecho de via”** (see Section 7.5).


## B1 — Targeted event deck
Use the v4 deterministic YAML deck (E-001..E-052) as the authoritative list and apply fairness constraints (7.3) + AutoChoice (7.4).

## B2 — Infrastructure modifiers
- INF-01 "Terminal del Metro anunciado"
- INF-02 "Nueva via expresa"

## B3 — Global shocks (capped)
Use the v4 list (G-01..G-06).

## B4 — Insurance mitigation tables (authoritative)
Incendio (E-045):
- NONE: cash −3, rent −2 (1), fame −1
- BASIC: cash −1, rent −1 (1), fame 0
- FULL: cash 0, rent −1 (1), fame 0

Structural/liability (E-046, E-047, E-050, E-052):
- BASIC: cash loss −1
- FULL: cash loss −2 and Fame loss −1 (min 0)

Small damage (E-048):
- BASIC/FULL cancel cash loss

Tenant lawsuit (E-050):
- FULL cancels all cash loss
- Compliance ≥2 cancels Fame loss

---

# Appendix C — UI acceptance criteria (screen-by-screen; deterministic)
(keep v4; update: add Team Set selector and Election/Vote UI state)

---

# Appendix D — Share assets (deterministic)
(keep v4; update: include dynasty disclaimer on share page footer)

---

# Appendix E — AI portrait pipeline (operational; low-cost; safe)
(keep v4)

---

# Appendix F — Networking protocol (authoritative)

## F1 Envelope
```json
{
  "v": 1,
  "type": "STRING",
  "roomId": "RID",
  "matchId": "MID",
  "clientId": "CID",
  "nonce": 123,
  "idempotencyKey": "uuid",
  "payload": {}
}
```

## F2 Required message types
Client → Server:
- JOIN_ROOM, LEAVE_ROOM
- SET_PROFILE
- SUBMIT_BID
- SUBMIT_ACTION
- SUBMIT_DEAL_OFFER, SUBMIT_DEAL_RESPONSE
- SUBMIT_MOOD_VOTE
- SUBMIT_ELECTION_VOTE
- SUBMIT_WATCHER_RUMOR
- SUBMIT_AWARD_VOTE
- REQUEST_SNAPSHOT

Server → Client:
- ROOM_STATE
- MATCH_START
- PHASE_CHANGE
- REVEAL_MARKET
- MARKET_SIGNALS_REVEAL
- MOOD_OPTIONS_REVEAL
- ELECTION_OPTIONS_REVEAL
- DEALS_UPDATE
- EVENT_RESULTS
- PRESS_TICKER
- ACHIEVEMENT_UNLOCK
- SHARE_ASSET_READY
- STATE_SNAPSHOT
- MATCH_END

Validation:
- strict per-phase checks
- reject duplicate nonce
- rate limit by session+IP
- server authoritative

---
# Appendix V — Visual Spec Addendum (production-ready)

## V1 Art direction — 3 pillars (young, Peruvian, memeable)
**Pillar 1: “Tabloid x Newspaper” UI**
- Match UI feels like a live **front page**: ticker, headlines, stamps, redactions, “exclusive” tags.
- Texture: subtle halftone, paper grain, ink bleed—lightweight.

**Pillar 2: “Chicha energy” accents**
- Chicha poster language for **stickers** and big moments: bursts, bold outlines.
- Modern restraint: chicha as **punctuation**, not constant background.

**Pillar 3: “Lima as a board”**
- Stylized Lima map with readable district shapes.
- Properties as pins with dynasty color rings + upgrade pips.

**Do-not-do**
- Photoreal property photos in match.
- Tiny text walls (headlines ≤70 chars; card bodies ≤120 chars).
- Heavy shaders/particles (web performance).

---

## V2 Match UI kit (Godot scenes/components)
Implement as reusable scenes with explicit states.

### V2.1 `PropertyCard.tscn`
**Elements**
- Title (nombre_es)
- District chip
- Tag icons row (max 3)
- BasePrice + BaseRent
- Upgrade pips (0–2)
- Insurance badge
- “OBRA” mini-badge if INF corridor active

**States**
- normal / hovered / selected / disabled
- sold overlay with stamp “ADJUDICADO”
- corridor overlay (thin animated line + “OBRA” label)

### V2.2 `BidPanel.tscn`
- Bid stepper + quick buttons (+1/+2)
- One-tap “Pujar”
- Fame tie-break token toggle
- Submitted lock state: “PUJA ENVIADA”

### V2.3 `ActionBar.tscn`
Buttons (icons + labels):
- Desarrollar, Asegurar, Seguridad, PR, Auditoria, Trato, Romper pacto (conditional), Pasar  
Error toast (Spanish): **“No te alcanza, causa.”**

### V2.4 `EventModal.tscn`
- Impact deltas (cash/fame/rent)
- Mitigation chips (“Seguro FULL”, “Seguridad activa”, “Tope de dano”)
- One big sticker for scandal: “AMPAY”
- One big stamp for bureaucracy: “OBSERVADO”

### V2.5 `PressTicker.tscn`
- Persona icon + 1–3 headlines (cycle every 2s)
- Tap opens a “Portada del round” overlay.

### V2.6 `MoodVoteWidget.tscn`
- Two big buttons; one tap; shows “VOTASTE”
- Optional live percentages (host option).

### V2.7 `ElectionVoteWidget.tscn`
- Two candidate cards (slogan + 1-line effect summary)
- Owner contribution: 0/1/2
- Watchers: one-tap vote

### V2.8 `MiniLeaderboard.tscn`
- Toggle: Riqueza / Fama
- Show top 5

---

## V3 MapView spec (stylized Lima; lightweight)
**Map style:** flat vector; crisp borders; soft shadow; minimal labels.

**District regions (10–14)**
Miraflores, San Isidro, Barranco, Surco, La Molina, Centro, Jesus Maria/Lince, Magdalena/San Miguel, Callao, Ate/SJL, Chorrillos, Independencia/Los Olivos.

**Pins**
- Dynasty color ring
- Small property-type icon (casa/oficina/almacen)
- Upgrade pips
- Insurance tiny shield overlay

**INF corridor**
- Animated line across 1–2 districts labeled “METRO” or “VIA”
- Expropriation moment: corridor flashes red + blueprint overlay.

---

## V4 Motion + microinteractions (youth appeal)
**Durations**
- UI transitions: 150–250ms
- Big moments: 400–600ms
- Timer warning: pulse every 1s in last 5s

**Signature animations**
- Auction win: stamp slam “ADJUDICADO” + subtle camera punch
- Extortion: quick shake + sticker “CUPO”
- Bureaucracy: stamp “OBSERVADO” + paper rustle
- Achievement: badge pop + light confetti (10–20 sprites max)
- Expropriation: “TRAZO” blueprint overlay + corridor flash

---

## V5 Audio (small set, high impact)
SFX (8): bid_submit, auction_win, cash_gain, cash_loss, stamp, scandal, achievement, timer_tick  
Optional voice stingers (short): “AMPAY!”, “ADJUDICADO!”, “AL TOQUE!”

---

## V6 Typography & colors (match vs admin)
- **Admin/menu:** Graf Design System.
- **Match typography:**
  - Headlines: Oswald or Bebas Neue
  - UI/body: Inter or Rubik
- Palette concept:
  - Paper off-white (#F6F3EE), ink black (#111), muted gray (#6B6B6B)
  - Accent: scandal red, stamp blue, money green, alert yellow (exact hex to be finalized)

---

## V7 Asset inventory (MVP counts)
- 1 Lima map base (SVG → rasterized)
- 12 district masks (SVG)
- 20 property-type icons (SVG)
- 60 badge icons (SVG)
- 10 press persona avatars
- 12 skyline backplates
- 2 corridor overlays (metro/via)
- 40 sticker shapes (AMPAY, CUPO, OBSERVADO, CON FE, YAPE, etc.)

---

## V8 Image-generator prompts (consistent style; non-photoreal)
**Skyline backplates (16:9):**
> “Stylized vector illustration of the skyline of {DISTRICT} in Lima, Peru. Modern flat design, clean shapes, subtle paper texture, minimal details, no photorealism, high contrast, young energetic vibe, slight halftone grain, warm off-white background like newspaper paper, limited color palette, no text, aspect ratio 16:9.”

**Press persona avatars (square):**
> “Cartoon editorial portrait avatar for a Peruvian news persona: {PERSONA}. Bold outlines, friendly satire, modern chicha-inspired accent colors, paper texture, not offensive, not based on a real person, clean background, square icon.”

**Sticker set (icons only):**
> “Set of 10 sticker-style badge illustrations with chicha poster energy: bold outlines, burst shapes, playful, modern, Peruvian vibe. Each sticker has an icon only (no text). Themes: scandal, bureaucracy stamp, money, yape, security, metro line, fire, earthquake, trophy, handshake. Flat vector style, paper texture.”

**Map base (concept only; final must be redrawn vector):**
> “Stylized simplified map of Lima metropolitan area divided into 12 district regions with clean borders and soft shadows. Minimal labels, flat vector style, paper texture, modern, game UI friendly, no real street accuracy required, high readability on mobile.”

---

## V9 Share assets (Portada + Trophy)
**Portada**
- Masthead “IMPERIO INMOBILIARIO”
- Winner hero portrait + dynasty crest + color
- 3–5 tabloid headlines + sticker accents
- QR + short link
- Mandatory disclaimer footer (Spanish)

**Trophy**
- Vertical 1080×1920
- One big sticker + one badge + one headline
- Winner name + team + district highlight
- Stamp animation on reveal
