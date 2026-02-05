# IMPERIO INMOBILIARIO — Spec Amendments v12.2

**Source:** Gaming Design Bible audit - MEDIUM issues (2026-02-04)
**Applies to:** `imperio_master_spec_v11.md` (v12.1)
**Scope:** All MEDIUM findings from the cross-reference audit

---

## CHANGELOG (v12.1 → v12.2)

### Property Deck (A7)
- P-06 rent: 45,000 → 46,000 (yield 12.86% → 13.14%)
- P-07 rent: 50,000 → 52,000 (yield 12.50% → 13.00%)
- P-06 district: Pueblo Libre → Jesus Maria/Lince (undefined district fix)
- P-13 district: Surquillo → Miraflores (undefined district fix)
- P-17: Added `turismo` tag (Miraflores is major tourist area)

### Event Deck (Appendix B)
- Added 8 new positive events (E-P04 through E-P11)
- New ratio: ~76% negative / ~20% positive (was 89%/7%)

### TypeScript Types (A1)
- Added 8 missing interfaces: WatcherState, AuctionBid, BotConfig, SmartDealSuggestion, InfraModifier, MarketSignal, ElectionState, ChallengeProgress

### Colyseus Schema (A9)
- Expanded ImperioState with 12 missing fields
- Added 5 new Schema classes: PackageSchema, DealSchema, MarketSignalSchema, MoodEffectSchema, InfraSchema

### Networking Protocol (F2-F3)
- Added 4 missing message types: TOGGLE_BUSY_MODE, SUBMIT_FAME_SPEND, SET_AUTO_PAY_PREFERENCE, AWARD_RESULTS
- Added 5 missing delta operations: SET_POLITICAL_EXPOSURE, SET_PROTECTED_UNTIL, SET_MOOD_ACTIVE, SET_METER, SET_TRADE_LOCK

---

# AMENDMENT M1: PROPERTY DECK FIXES (A7)

**Action:** [AMEND] Appendix A7 property entries

### P-06 Fix
```
P-06 | Depas Pueblo Libre | Jesus Maria/Lince | residencial, cultura | 350,000 | 46,000
```
- Changed district from "Pueblo Libre" to "Jesus Maria/Lince" (geographically adjacent, valid district)
- Changed rent from 45,000 to 46,000 (yield: 13.14%)

### P-07 Fix
```
P-07 | Taller Automotriz Chorrillos | Chorrillos | logistica, costa | 400,000 | 52,000
```
- Changed rent from 50,000 to 52,000 (yield: 13.00%)

### P-13 Fix
```
P-13 | Edificio Surquillo | Miraflores | residencial | 750,000 | 75,000
```
- Changed district from "Surquillo" to "Miraflores" (geographically adjacent, valid district)

### P-17 Fix
```
P-17 | Depa Vista Mar Miraflores | Miraflores | residencial, costa, turismo, premium_area | 1,000,000 | 90,000
```
- Added `turismo` tag (Miraflores is Lima's primary tourist district)

---

# AMENDMENT M2: NEW POSITIVE EVENTS (Appendix B)

**Action:** [INSERT] After E-P03 in Appendix B (Positive Events section)

```markdown
### E-P04: Contrato de seguridad exitoso
**Category:** SEGURIDAD
**Type:** TARGETED_POSITIVE
**Condition:** `owner.hasSecurityContract`
**Effects:**
```yaml
cash: 80000
fame: 1
```
**Headlines:**
- "Seguridad de {PROPERTY} frustra asalto"
- "Guardias de {OWNER} capturan delincuentes"

---

### E-P05: Inspección impecable
**Category:** LICENCIAS
**Type:** TARGETED_POSITIVE
**Condition:** `owner.compliance >= 2`
**Effects:**
```yaml
cash: 60000
compliance: 1
```
**Headlines:**
- "Municipio felicita a {OWNER} por cumplimiento"
- "{PROPERTY} pasa auditoría con nota perfecta"

---

### E-P06: Inversión extranjera
**Category:** INCIDENTES
**Type:** TARGETED_POSITIVE
**Condition:** `property.tags.includes('oficinas') || property.tags.includes('logistica')`
**Effects:**
```yaml
cash: 100000
rentDelta: 15000
rentDuration: 2
```
**Headlines:**
- "Multinacional elige {PROPERTY} como sede regional"
- "Inversión millonaria llega a {DISTRICT}"

---

### E-P07: Viral positivo
**Category:** PRENSA
**Type:** TARGETED_POSITIVE
**Effects:**
```yaml
fame: 2
rentDelta: 10000
rentDuration: 1
```
**Headlines:**
- "{PROPERTY} se vuelve tendencia por razones buenas"
- "Influencer elogia a {OWNER} y se hace viral"

---

### E-P08: Donación reconocida
**Category:** PRENSA
**Type:** TARGETED_POSITIVE
**Condition:** `owner.fame >= 3`
**Effects:**
```yaml
fame: 2
politicalExposure: -1
```
**Headlines:**
- "{OWNER} recibe premio por filantropía"
- "Fundación {TEAM} inaugura comedor popular"

---

### E-P09: Vecinos agradecidos
**Category:** SEGURIDAD
**Type:** TARGETED_POSITIVE
**Effects:**
```yaml
fame: 1
rentDelta: 8000
rentDuration: 1
```
**Headlines:**
- "Junta vecinal agradece a {OWNER}"
- "Comerciantes de {DISTRICT} celebran seguridad"

---

### E-P10: Exoneración tributaria
**Category:** LICENCIAS
**Type:** TARGETED_POSITIVE
**Condition:** `property.tags.includes('cultura') || property.tags.includes('gobierno')`
**Effects:**
```yaml
cash: 120000
```
**Headlines:**
- "{PROPERTY} declarado bien de interés cultural"
- "SUNAT otorga beneficio fiscal a {OWNER}"

---

### E-P11: Contrato gubernamental
**Category:** INCIDENTES
**Type:** TARGETED_POSITIVE
**Condition:** `property.tags.includes('gobierno') || property.tags.includes('oficinas')`
**Effects:**
```yaml
cash: 100000
rentDelta: 20000
rentDuration: 2
```
**Headlines:**
- "Ministerio alquila {PROPERTY} por 5 años"
- "Gobierno firma contrato millonario con {OWNER}"
```

**Updated Event Balance:**
| Category | Negative | Positive | Total |
|----------|----------|----------|-------|
| SEGURIDAD | 8 | 2 | 10 |
| LICENCIAS | 10 | 3 | 13 |
| PRENSA | 10 | 3 | 13 |
| INCIDENTES | 8 | 3 | 11 |
| GLOBAL | 5 | 0 | 5 |
| **TOTAL** | **41** | **11** | **52** |

---

# AMENDMENT M3: MISSING TYPESCRIPT INTERFACES (A1)

**Action:** [INSERT] At end of Appendix A1

```typescript
// === Watcher State (Section 3.3) ===
/** State for a Watcher participant (non-Owner viewer with limited influence) */
interface WatcherState {
  watcherId: Id;
  handle: string;
  displayName: string;
  isAnonymous: boolean;
  rumorUsed: boolean;
  bettingState: WatcherBettingState;
  awardVotes: Record<string, Id>;
  watcherXP: number;
  watcherBadge?: "OBSERVADOR" | "ANALISTA" | "EXPERTO";
}

interface WatcherBettingState {
  predictedWinnerId: Id | null;
  propertyId: Id | null;
  wasCorrect?: boolean;
}

// === Auction Bid (Section 5.5) ===
/** Pending bid during BID phase (sealed, simultaneous) */
interface AuctionBid {
  ownerId: Id;
  propertyId: Id;
  amount: number;
  timestamp: number;
  fameTokenUsed: boolean;
}

// === Bot Config (Appendix A8) ===
/** Configuration for bot behavior (Tutorial/Autopilot/fill seats) */
interface BotConfig {
  difficulty: "EASY" | "NORMAL" | "HARD";
  cashReserve: number;
  bidAggressiveness: number;
  isTutorial: boolean;
  canInitiateDeals: boolean;
  evaluatesDeals: boolean;
}

// === Smart Deal Suggestion (Section 7.2.6) ===
/** Server-generated contextual deal suggestion */
interface SmartDealSuggestion {
  template: DealTemplateId;
  targetOwnerId: Id;
  suggestedAmount?: number;
  propertyId?: Id;
  packageId?: Id;
  reason: "COMPLETES_DISTRICT_SET" | "LOW_CASH_NEEDS_LOAN" | "COMPETING_FOR_DISTRICT" | "PACKAGE_SALE_OPPORTUNITY";
  label_es: string;
}

// === Infrastructure Modifier (Section 8.5) ===
/** Infrastructure corridor modifier affecting a district */
interface InfraModifier {
  id: Id;
  type: "METRO_TERMINAL" | "VIA_EXPRESA";
  name_es: string;
  affectedDistricts: string[];
  rentDelta: number;
  worthDelta: number;
  duration: number;
  untilRound: number;
  enablesExpropriation: boolean;
}

// === Market Signal (Section 6.4) ===
/** Market signal affecting rent by economic tags */
interface MarketSignal {
  id: Id;
  name_es: string;
  affectedTags: EconomicTag[];
  tagMods: Record<string, number>;
  active: boolean;
  activatedRound?: number;
}

// === Election State (Sections 5.10, Part 9) ===
/** Per-match election state */
interface ElectionState {
  electionRound: number;
  candidates: Candidate[];
  contributions: Record<Id, Record<Id, number>>;
  watcherVotes: Record<Id, number>;
  winnerId?: Id;
  bundleDuration: number;
  effectsUntilRound?: number;
  isMiniElection: boolean;
}

// === Challenge Progress (Section 3.5.6) ===
/** Progress tracking for daily/weekly/seasonal challenges */
interface ChallengeProgress {
  challengeId: Id;
  tier: "DAILY" | "WEEKLY_DYNASTY" | "SEASONAL";
  progress: number;
  target: number;
  completed: boolean;
  completedAt?: number;
  rewardGD: number;
  bonusReward?: string;
}

interface DynastyChallengeProgress extends ChallengeProgress {
  tier: "WEEKLY_DYNASTY";
  dynastyId: Id;
  memberContributions: Record<Id, number>;
}
```

---

# AMENDMENT M4: EXPANDED COLYSEUS SCHEMA (A9)

**Action:** [REPLACE] The ImperioState schema section in A9

```typescript
// === NEW SCHEMA CLASSES ===

export class PackageSchema extends Schema {
  @type("string") packageId: string;
  @type("string") ownerId: string;
  @type("string") category: string;
  @type(["string"]) propertyIds = new ArraySchema<string>();
  @type("number") createdRound: number;
  @type("number") rentBonusPerProperty: number = 6_000;
  @type("number") worthBonus: number = 75_000;
  @type("number") dissolvedRound: number = 0;
}

export class DealSchema extends Schema {
  @type("string") dealId: string;
  @type("string") templateId: string;
  @type("string") fromOwnerId: string;
  @type("string") toOwnerId: string;
  @type("number") createdRound: number;
  @type("string") status: string = "PENDING";
  @type("string") fieldsJson: string = "{}";
  @type("string") counterFieldsJson: string = "";
  @type("number") scheduledRepayAmount: number = 0;
  @type("number") scheduledRepayRound: number = 0;
}

export class MarketSignalSchema extends Schema {
  @type("string") id: string;
  @type("string") nombre_es: string;
  @type("string") tagModsJson: string = "{}";
}

export class MoodEffectSchema extends Schema {
  @type("string") id: string;
  @type("string") nombre_es: string;
  @type("string") effectType: string;
  @type("number") delta: number = 0;
  @type("number") multiplier: number = 1;
  @type("string") sentiment: string = "NEUTRAL";
}

export class InfraSchema extends Schema {
  @type("string") infraId: string;
  @type("string") district: string;
  @type("number") untilRound: number;
  @type("number") rentDelta: number = 5_000;
  @type("number") worthDelta: number = 100_000;
  @type("boolean") enablesExpropriation: boolean = false;
}

// === EXPANDED ImperioState ===

export class ImperioState extends Schema {
  // Core
  @type("string") roomId: string;
  @type("string") mode: string = "STANDARD";
  @type("string") phase: string = "LOBBY";
  @type("number") round: number = 0;
  @type("number") phaseDeadline: number = 0;

  // Entities
  @type({ map: OwnerSchema }) owners = new MapSchema<OwnerSchema>();
  @type({ map: PropertySchema }) properties = new MapSchema<PropertySchema>();
  @type({ map: PackageSchema }) packages = new MapSchema<PackageSchema>();
  @type({ map: DealSchema }) deals = new MapSchema<DealSchema>();

  // Market
  @type(["string"]) marketRow = new ArraySchema<string>();
  @type({ map: MarketSignalSchema }) marketSignals = new MapSchema<MarketSignalSchema>();

  // Mood
  @type(["string"]) moodOptions = new ArraySchema<string>();
  @type("string") activeMoodId: string = "";
  @type(MoodEffectSchema) activeMoodEffect: MoodEffectSchema;

  // Election
  @type("boolean") electionActive: boolean = false;
  @type(["string"]) electionCandidates = new ArraySchema<string>();

  // Spectators
  @type("number") watcherCount: number = 0;

  // Infrastructure
  @type({ map: InfraSchema }) infra = new MapSchema<InfraSchema>();

  // Meters
  @type("string") metersJson: string = "{}";

  // Press
  @type("string") pressPersonaId: string = "";
  @type(["string"]) pressTicker = new ArraySchema<string>();
}
```

---

# AMENDMENT M5: MISSING MESSAGE TYPES (F2)

**Action:** [INSERT] Into F2 message type tables

### Client → Server (Add to table)

| Type | Payload | Valid Phases |
|------|---------|--------------|
| TOGGLE_BUSY_MODE | `{ enabled: boolean }` | Any |
| SUBMIT_FAME_SPEND | `{ spendType: FameSpendType, eventId?: string }` | EVENTS |
| SET_AUTO_PAY_PREFERENCE | `{ preferAutoPay: boolean }` | LOBBY, ACTION |

### Server → Client (Add to table)

| Type | Payload | When Sent |
|------|---------|-----------|
| AWARD_RESULTS | `{ awards: AwardResult[] }` | After AWARDS phase |

### Type Definitions

```typescript
type FameSpendType = "YAPE_DE_EMERGENCIA" | "CONTROL_DE_DANOS" | "CORTINA_DE_HUMO";

interface AwardResult {
  awardId: string;
  winnerId: Id;
  winnerHandle: string;
  voteCount: number;
  totalVoters: number;
}
```

---

# AMENDMENT M6: MISSING DELTA OPERATIONS (F3)

**Action:** [INSERT] Into F3 operations table

| Op | Target | Value | Description |
|----|--------|-------|-------------|
| SET_POLITICAL_EXPOSURE | ownerId | 0-3 | Election contribution tracking |
| SET_PROTECTED_UNTIL | ownerId | round# | Security Escort protection |
| SET_MOOD_ACTIVE | "-" | MoodId | Currently active mood effect |
| SET_METER | meterId | number | World meter value |
| SET_TRADE_LOCK | propertyId | round# | Property trade restriction |

### Examples

```json
{ "op": "SET_POLITICAL_EXPOSURE", "id": "O_01", "val": 2 }
{ "op": "SET_PROTECTED_UNTIL", "id": "O_03", "val": 7 }
{ "op": "SET_MOOD_ACTIVE", "id": "-", "val": "M-03" }
{ "op": "SET_METER", "id": "InterestRates", "val": 2 }
{ "op": "SET_TRADE_LOCK", "id": "P_12", "val": 5 }
```

---

# SUMMARY

| Amendment | Section | Changes |
|-----------|---------|---------|
| M1 | A7 | P-06, P-07, P-13 district/rent fixes; P-17 turismo tag |
| M2 | Appendix B | 8 new positive events (E-P04 to E-P11) |
| M3 | A1 | 8 new TypeScript interfaces |
| M4 | A9 | 5 new Schema classes, expanded ImperioState |
| M5 | F2 | 4 new message types |
| M6 | F3 | 5 new delta operations |

**Version note:** Applying all amendments advances the spec to v12.2.

---

*End of Spec Amendments v12.2*
