# Imperio Inmobiliario — Pending Tasks

**Last Updated:** 2026-02-05

---

## Critical Path (Blocking)

### [x] Phase 2: Deep Code Audit of Open Source Candidates
**Status:** COMPLETED (2026-02-05)
**Deliverable:** Updated `open-source-foundation-research.md`

**Results Summary:**
1. **sominator/colyseus-2d-card-game-templates** — SKIP (UNLICENSED, Godot 3.x, no value)
2. **alpapaydin/Godot4-Multiplayer-Survivor-IO-Game** — USE FOR PATTERNS (MIT, WebSocket/SSL, exports)
3. **chun92/card-framework** — HIGHLY RECOMMENDED (MIT, Godot 4.5+, excellent card UI)

**Recommendations:**
- Card UI: Use chun92/card-framework
- WebSocket/SSL: Extract patterns from alpapaydin
- Server: Use official Colyseus starter (npm create colyseus-app)

---

### [x] Spec Amendment v12.4: Open Source Foundation Commitments
**Status:** COMPLETED (2026-02-05)
**Blocked by:** Nothing (Phase 2 complete)
**Blocks:** Implementation plan update

**Tasks:**
- [x] Promote Appendix O from "evaluation" to "directive"
- [x] Add "Reuse Strategy" section specifying what's copied vs. custom
- [x] Expand Section 11.2 (Client Architecture) with scene structure
- [x] Add Appendix O4: "Reuse Matrix"
- [x] Update A9 to reference actual SDK/template code (deferred to implementation)
- [x] Review with gaming-design-bible skill for anti-pattern check (completed in code audit)

**Deliverable:** `spec_amendments_v12_4.md` (APPLIED to master spec)

---

### [ ] Expanded Implementation Plan: Full Game (Server + Client)
**Status:** NOT STARTED
**Blocked by:** Spec amendment v12.4
**Blocks:** Implementation start

**Tasks:**
- [ ] Add Phase 7-10: Godot client infrastructure
- [ ] Add Phase 11-12: Integration and E2E testing
- [ ] Define open source fork points in each phase
- [ ] Add asset pipeline tasks
- [ ] Add mobile-specific tasks (touch, responsive)
- [ ] Add HTML5/WebAssembly export tasks
- [ ] Update testing strategy for client

**Deliverable:** Updated `modular-implementation-plan.md` or new `full-implementation-plan.md`

---

## Backlog (Non-Blocking)

### [ ] Evaluate Alternative: boardgame.io for Rapid Prototype
**Priority:** LOW
**Notes:** Could build a playable prototype faster with boardgame.io before committing to Colyseus+Godot. Trade-off: throwaway code vs. faster validation.

### [ ] Research: Existing Godot Card Game Templates on Asset Library
**Priority:** LOW
**Notes:** May find additional card UI options beyond chun92/card-framework.

### [ ] Research: Colyseus Cloud vs. Self-Hosted
**Priority:** MEDIUM
**Notes:** Colyseus offers managed hosting. Evaluate cost vs. self-hosting complexity.

---

## Completed

### [x] Spec Amendment v12.4: Open Source Foundation Commitments
**Completed:** 2026-02-05
**Deliverable:** `spec_amendments_v12_4.md` applied to `imperio_master_spec_v11.md`
**Summary:**
- Section 1.5: Commitments replacing Strategy (reuse matrix)
- Section 11.2: Expanded client architecture (250+ lines)
- Appendix O: Directives replacing Evaluation (O1-O5)

### [x] Phase 2: Deep Code Audit of Top Candidates
**Completed:** 2026-02-05
**Deliverable:** Updated `plans/open-source-foundation-research.md`
**Summary:**
- sominator/colyseus-card-game: SKIP (UNLICENSED, Godot 3.x)
- alpapaydin/Godot4-Multiplayer: USE FOR PATTERNS (WebSocket/SSL)
- chun92/card-framework: USE FOR CARD UI (excellent quality, MIT)

### [x] Phase 1: Web Research on Open Source Foundations
**Completed:** 2026-02-05
**Deliverable:** `plans/open-source-foundation-research.md`
**Summary:** Identified 10 candidate repositories across 3 tiers. No single repo covers full stack. Reuse estimate: 40-60% infrastructure.

### [x] CLAUDE.md Improvements
**Completed:** 2026-02-05
**Changes:** Added Critical Files table, Planned Module Structure, Development Commands, Available Skills section.

---

## Decision Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-02-05 | Spec changes before plan changes | Spec defines WHAT/WHY, plan defines HOW/WHEN. Proper sequence. |
| 2026-02-05 | Code audit before spec commitment | Avoid committing to repos with hidden issues in the spec. |
