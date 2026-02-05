# IMPERIO INMOBILIARIO — Spec Amendments v12.3

**Source:** Gaming Design Bible audit - LOW issues (2026-02-04)
**Applies to:** `imperio_master_spec_v11.md` (v12.2)
**Scope:** All LOW findings from the cross-reference audit

---

## CHANGELOG (v12.2 → v12.3)

### Visual Ledger
- Added adaptive timing: base 3.0s + 0.15s per item, max 5.0s
- Added item limit (12 items max, overflow collapses to summary)

### Audio System
- Added phase-based dynamic audio layers (V6.1 NEW)
- Added tension escalation system for BID/EVENTS phases
- Added ambient soundscapes per phase

### GDScript Examples (A9)
- Removed deprecated Godot 3.x ColyseusClient example
- Kept only Godot 4.x compliant ImperioClient and VisualLedger examples

### Currency Clarification
- Removed all Dynasty XP references (deprecated)
- Added explicit note: GD is the only currency for all progression

---

# AMENDMENT L1: VISUAL LEDGER ADAPTIVE TIMING

**Action:** [REPLACE] Section 5.7 and C3

### 5.7 VISUAL_LEDGER Phase (REVISED in v12.3)

**Purpose**: Visualize the "Invisible Accountant" calculations

**Duration**: Adaptive (base 3.0s + 0.15s per item, capped at 5.0s max)

**Formula**:
```
duration = min(3.0 + (numItems × 0.15), 5.0)
```

**Item Limit**: Maximum 12 line items displayed. If more items exist:
- Group minor items (< GD 5,000) into "Otros ingresos" / "Otros costos"
- Always show individual rent per property if ≤ 5 properties
- Properties 6+ collapse to "Rentas adicionales (×N): +GD X"

**Client Behavior**:
- Lock ACTION input buttons
- Display modal: Income Receipt animation
- Server proceeds after calculated duration regardless of client animation state

### C3: Visual Ledger Animation (REVISED in v12.3)

**Trigger**: INCOME_CALC → VISUAL_LEDGER phase transition

**Duration**: Adaptive (see 5.7)

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

**Animation Sequence** (times relative to item count):
```
t=0.0s              Modal appears with header
t=0.5s              First income item fades in
t=0.5s + n×0.15s    Subsequent income items (stagger)
t=midpoint          Cost items begin (stagger 0.12s each)
t=duration-1.0s     Total appears with scale emphasis
t=duration-0.5s     Cash counter animates to final value
t=duration          Modal fades out, ACTION phase begins
```

**Overflow Handling**:
When more than 12 items exist, collapse to summary groups:
```
┌────────────────────────────────────────┐
│   Rentas (×8):          +GD 180,000 🟢 │
│   Bonos y señales:       +GD 14,000 🟢 │
│   ─────────────────────────────────    │
│   Costos totales:       -GD 32,000 🔴  │
│   ─────────────────────────────────    │
│   TOTAL:               +GD 162,000     │
└────────────────────────────────────────┘
```

**Constraint**: All input buttons disabled during animation. Server-side timer is authoritative.

---

# AMENDMENT L2: DYNAMIC PHASE-BASED AUDIO (V6.1 NEW)

**Action:** [INSERT] After V6 in Appendix V

## V6.1: Dynamic Audio Layers (NEW in v12.3)

Audio should respond to game state, not just play static tracks. This creates emotional peaks at key moments.

### Phase-Based Ambient Layers

| Phase | Base Track | Layer | Intensity |
|-------|-----------|-------|-----------|
| LOBBY | music_menu | None | Low |
| REVEAL | music_match | market_ambient | Low |
| BID | music_match | auction_tension | Escalating |
| INCOME_CALC | music_match | None (brief) | Low |
| VISUAL_LEDGER | music_match | ledger_shuffle | Medium |
| ACTION | music_match | thinking_ambient | Low |
| MOOD | music_match | crowd_murmur | Medium |
| VOTE | music_match | political_ambient | Medium |
| DEALS | music_match | negotiation_ambient | Low |
| EVENTS | music_match | suspense_build | Escalating |
| PRESS | music_match | newsroom_ambient | Medium |
| AWARDS | music_victory | celebration_layer | High |

### Tension Escalation System

**BID Phase Escalation**:
```
Timer > 15s remaining: Normal tempo
Timer 10-15s: +10% tempo, add subtle bass pulse
Timer 5-10s: +20% tempo, heartbeat layer fades in
Timer < 5s: Heartbeat dominant, high-pass filter on music
Timer = 0: Gavel SFX cuts all layers
```

**EVENTS Phase Escalation**:
```
Event card revealed: Suspense sting
Target selection animation: Drum roll layer
Target revealed (self): Impact sting + bass drop
Target revealed (other): Relief sigh SFX
```

### Audio Layer Assets (6 NEW)

| ID | Layer | Description |
|----|-------|-------------|
| layer_auction_tension | BID | Ticking clock, rising strings |
| layer_heartbeat | BID (final 5s) | Rhythmic heartbeat, 90 BPM |
| layer_suspense_build | EVENTS | Low drone, intermittent stings |
| layer_crowd_murmur | MOOD/VOTE | Crowd ambience, occasional cheer |
| layer_negotiation | DEALS | Paper shuffle, pen scratches |
| layer_celebration | AWARDS | Crowd cheering, confetti rustle |

### Implementation Notes

- All layers should be separate audio files that crossfade
- Use AudioStreamPlayer with bus routing for layer mixing
- Tempo changes via AudioServer pitch_scale (not re-sampling)
- Heartbeat layer: Pre-rendered at 90 BPM, loop seamlessly
- Target: 3-5 simultaneous layers max for mobile performance

### Volume Ducking Rules

| Event | Duck Target | Amount | Duration |
|-------|-------------|--------|----------|
| SFX plays | Music | -6dB | 0.3s + SFX length |
| Voice stinger | Music + SFX | -9dB | Stinger length |
| Phase transition | All layers | -3dB | 0.5s fade |
| Tension peak | Ambient layers | -12dB | Until resolution |

---

# AMENDMENT L3: REMOVE DEPRECATED GODOT 3.X CODE (A9)

**Action:** [DELETE] The first GDScript example in A9 (lines 2779-2842)

The deprecated `WebSocketClient` and Godot 3.x signal connection syntax should be removed. Only the Godot 4.x compliant examples remain:
- `ImperioClient` (line 4661+) — Uses `WebSocketPeer`, typed signals
- `VisualLedger` (line 4789+) — Uses `@onready`, typed arrays, `create_tween()`

**Rationale:** Godot 3.x reached end-of-life. All examples should use Godot 4.x syntax to avoid confusion during implementation.

### Code to DELETE (A9 Section "Godot Client Connection"):

```gdscript
# client/scripts/network/ColyseusClient.gd  ← DELETE THIS ENTIRE BLOCK
extends Node

var client: WebSocketClient
var room_state: Dictionary = {}
# ... (entire example through line ~2842)
```

**Replace with note:**
> **Note:** For Godot client implementation, see the `ImperioClient` class in the "Appendix A9: Full Code Examples" section below, which uses Godot 4.x `WebSocketPeer` API.

---

# AMENDMENT L4: REMOVE DYNASTY XP REFERENCES

**Action:** [AMEND] Multiple sections to remove Dynasty XP references

### Changelog Line 42
**Old:**
> - **Referral rework** (3.5.4): Multi-tier GD rewards (25K/50K/100K) replacing single-tier Dynasty XP

**New:**
> - **Referral rework** (3.5.4): Multi-tier GD rewards (25K/50K/100K)

### Add Currency Clarification (Part 13, Section 13.1)

**Insert after opening paragraph:**

> **Currency Note (v12.3):** Graf Dollar (GD) is the ONLY progression currency. There is no separate "Dynasty XP" or experience point system. All rewards, rankings, and progression use GD exclusively. This simplifies the economy and prevents confusion between multiple currency types.

### Dynasty Section (3.4) — Add Clarification

**Insert at end of 3.4:**

> **Note:** Dynasty rankings are determined by aggregate GD wealth of members, not a separate XP system. Dynasty challenges reward GD directly to participating members.

---

# SUMMARY

| Amendment | Section | Changes |
|-----------|---------|---------|
| L1 | 5.7, C3 | Adaptive Visual Ledger timing (3.0s-5.0s), item overflow handling |
| L2 | V6.1 (NEW) | Phase-based dynamic audio layers, tension escalation |
| L3 | A9 | Remove Godot 3.x deprecated example |
| L4 | Multiple | Remove Dynasty XP references, clarify GD-only currency |

**Version note:** Applying all amendments advances the spec to v12.3.

---

*End of Spec Amendments v12.3*
