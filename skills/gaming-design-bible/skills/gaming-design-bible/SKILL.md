---
name: Gaming Design Bible
description: >-
  This skill should be used when the user asks to "design a game feature",
  "balance game economy", "add virality mechanics", "review game design",
  "evaluate monetization strategy", "design a leaderboard", "create retention loops",
  "design matchmaking", "review auction mechanics", "add social features to a game",
  "check game anti-patterns", "design a reward system", "review player onboarding",
  "design a progression system", or discusses multiplayer game architecture,
  turn-based game design, mobile game UX, player progression systems, or
  competitive game balance. Provides gaming industry best practices, competitive
  analysis from 15+ successful titles, virality engineering, anti-pattern avoidance,
  and framework-specific guidance for Colyseus + Godot 4.x + PostgreSQL stacks.
version: 1.0.0
---

# Gaming Design Bible

Comprehensive game design knowledge for multiplayer competitive games, with deep focus on turn-based economic/property games, mobile-first architecture, and viral growth mechanics.

## Core Design Pillars

Every design decision must pass through these five filters:

| Pillar | Question | Failure Mode |
|--------|----------|-------------|
| **Fairness** | Can a new player compete on equal footing? | Pay-to-win, snowball leaders |
| **Tension** | Does every round have a meaningful decision? | Dead states, autopilot turns |
| **Shareability** | Would a player screenshot/record this moment? | Forgettable outcomes |
| **Session Respect** | Does the match fit the player's available time? | Endless Monopoly syndrome |
| **Comeback Possibility** | Can last place win on the final round? | Runaway leader, early quit |

## Economy Design Principles

### Currency Hierarchy
Separate in-match economy from persistent progression:
- **Match currency** — Equal start, resets every game. Drives fair competition.
- **Prestige currency** — Accumulated from match outcomes. Drives retention and leaderboards.
- **Premium currency** (if applicable) — Cosmetic only. Never gameplay advantage.

Violating this separation kills competitive integrity. Reference: Clash Royale card levels controversy (P2W perception), Diablo Immortal backlash ($110K to max a character).

### Yield Curves
Property/asset games need deliberate yield spread:
- Entry assets: High yield % (13-15%), low absolute return — accessible, satisfying early game
- Premium assets: Low yield % (6-8%), high absolute return — aspirational, late-game power
- This forces strategic choice: many cheap assets (portfolio breadth) vs. few expensive ones (concentration)
- Reference: Power Grid's plant market creates identical tension

### Catch-Up Mechanics (Mandatory)
Every competitive game needs rubber-banding:
- **Leader friction** — Tax or cost increase for wealth leader (OpsCost scaling, leader tax)
- **Consolation mechanics** — Small bonuses for trailing players (R1 subsidies, cheaper loans)
- **Fairness caps** — Maximum damage per player per round (prevents elimination by bad luck)
- **Event targeting** — Higher probability of negative events for leaders (weighted RiskScore)
- Reference: Power Grid's turn-order reversal, Catan's robber targets leader, Mario Kart's blue shell

## Virality Engineering

### Share-Worthy Moment Design
Architect specific moments that trigger sharing:

1. **Dramatic reversals** — Come-from-behind victories via late auction wins or lucky event avoidance
2. **Auction climaxes** — Final-second bids that steal properties (countdown tension)
3. **Dynasty milestones** — League promotions, first Emperor badge, wealth records
4. **Betrayal moments** — Deal-breaking, non-compete violations, strategic backstabs
5. **Portfolio reveals** — End-match wealth calculations with suspenseful animation

### Portada (Share Card) Design
Auto-generated end-of-match share images must include:
- Player name + dynasty branding
- Match outcome (rank, final wealth, key stats)
- One "highlight moment" (biggest deal, best property, closest bid)
- Visual style matching game aesthetic
- QR code or deep link to download/play
- Reference: Brawl Stars battle log sharing, Chess.com game analysis cards

### Social Loop Architecture
Design a closed viral loop: Play → Share Moment → Friend Sees → Friend Downloads → Plays.
- Dynasty recruitment as viral driver (collective leaderboard)
- "Play with friends" must be frictionless (share room code, deep link)
- Spectator mode feeds streaming/content creation pipeline

For detailed virality patterns, share card specifications, and social hook mechanics, consult **`references/virality-patterns.md`**.

## Anti-Patterns (Hard "No-Go" List)

### Critical Anti-Patterns — Never Allow

| Anti-Pattern | Why It Kills Games | Solution |
|-------------|-------------------|----------|
| **Pay-to-Win** | Destroys competitive integrity, triggers refund waves | Cosmetic-only monetization, equal match starts |
| **Player Elimination** | Removed players leave and never return | Keep all players active until final round |
| **Snowball/Runaway Leader** | Once ahead, always ahead = boring for 5 of 6 players | Leader friction, fairness caps, catch-up mechanics |
| **Free-Text Chat** | Toxicity, moderation costs, legal liability | Quick Chat presets only (Rocket League model) |
| **Session > 20min (mobile)** | Players abandon mid-match, never return | Quick Mode (7 rounds), clear time commitment upfront |
| **Unfair RNG without mitigation** | "I lost because of luck" = uninstall | Insurance mechanics, fairness caps, pity timers |
| **Kingmaking** | Losing player decides winner via spite deals | Template-only deals, server-validated fairness checks |
| **Dead States** | No meaningful choices but game continues | Forced sale at 80% (prevents bankruptcy spiral), consolation mechanics |

### Warning Anti-Patterns — Handle Carefully

| Pattern | Risk | Mitigation |
|---------|------|------------|
| Analysis Paralysis | Turns too slow, other players bored | Phase timers (15-30s), Major/Minor action split |
| Information Overload | Mobile UI clutter, cognitive overwhelm | Progressive disclosure, visual ledger animations |
| Reward Fatigue | Daily rewards become chores | Variable ratio rewards, seasonal refresh |
| Onboarding Cliff | Complex rules lose players in 5 min | Bot tutorial match, progressive complexity |

For comprehensive anti-pattern analysis with 12 categories, real game failure case studies, and proven solutions, consult **`references/anti-patterns.md`**.

## Similar Games — Key Lessons

### Property/Economic Games
- **Monopoly**: Auction mechanic works; player elimination and endless sessions don't. Cap game length.
- **Monopoly GO!**: Sticker metagame + social sharing drove $2B+. Session pacing (2-3 min bursts). Dice rolls as dopamine hits.
- **For Sale**: Two-phase auction elegance (bid on properties, then sell for profit). Clean 20-min sessions.
- **Power Grid**: Turn-order reversal (leader goes last) is the gold standard for catch-up. Resource market pricing creates emergent strategy.

### Strategy/Engine-Building
- **Terraforming Mars**: Tag synergies (like property tags). Engine building via card combos. Late-game power curves.
- **Wingspan**: Major/Minor action split keeps decisions fast. Egg economy = simple but deep.
- **Brass Birmingham**: Network bonuses reward geographic concentration (like district set bonuses).

### Competitive Mobile/Live-Service
- **Clash Royale**: 2-min matches, trophy leagues, emote communication, spectator replays. Gold standard for mobile competitive.
- **Fall Guys**: Show format creates dramatic arcs. Crown economy as prestige currency. Costume cosmetics.
- **Among Us**: Viral via streaming. Social deduction simplicity. Emergency meetings as dramatic beats.
- **Chess.com**: ELO display drives engagement. Post-game analysis adds learning value. Daily puzzles for retention.

For detailed per-game analysis with specific mechanics to adopt/avoid and retention loop breakdowns, consult **`references/similar-games-analysis.md`**.

## Framework Guidance

### Colyseus (Server)
- Use `@colyseus/schema` with flat state trees — deep nesting kills serialization performance
- Phase-based games: implement state machine with explicit `currentPhase` enum and `phaseTimer`
- Reconnection: 60s grace period with bot autopilot (never kick mid-match)
- Room sizing: cap at 20 participants (12 Owners + 8 Watchers) for state sync efficiency
- Deterministic logic: use seeded random with server secret (never exposed to client)
- Matchmaking: separate queues for Quick/Standard/Long modes, ELO-band matching for ranked

### Godot 4.x (Client)
- WebSocket connection to Colyseus via WebAssembly export
- Scene-per-phase architecture: `LobbyScene → MatchScene → ResultsScene`
- Mobile text input: HTML DOM overlay (Godot virtual keyboard is unreliable on mobile web)
- Animations: Visual Ledger (cash flow) must complete before next phase (4.0s server wait)
- Audio: Dynamic layer system — base ambient + event stings + UI feedback

### PostgreSQL (Persistence)
- Match snapshots as JSONB (flexible schema, easy querying)
- Double-entry ledger for virtual currency (credit + debit per transaction, idempotency keys)
- Leaderboard queries: materialized views refreshed every 5 min, sorted sets in Redis for real-time

### Redis (Real-Time)
- Sorted sets for leaderboard queries (ZADD/ZRANGEBYSCORE)
- Rate limiting: sliding window counters per user per action type
- Pub/sub for cross-room events (dynasty notifications, friend online status)

For detailed framework patterns, code examples, scaling strategies, and common pitfalls, consult **`references/gaming-frameworks.md`**.

## Retention Loop Design

### Daily Loop (Login → Play → Reward)
- Daily challenges tied to in-match actions (not grind)
- Login streak with escalating but non-essential rewards
- "Play 1 Quick Match" as minimum engagement unit (7 min)

### Weekly Loop (Progress → Compare → Compete)
- Weekly leaderboard snapshots
- Dynasty challenges (collective goals)
- Map/event rotation for freshness

### Seasonal Loop (Rank → Decay → Climb)
- Monthly soft reset: balances above threshold decay 10%
- Seasonal exclusive cosmetics for top 100
- New game content (properties, events, candidates) each season
- Returning player catchup: bonus GD for first 3 matches after absence

## Design Review Checklist

When evaluating any game feature or change, verify:

- [ ] **Fairness**: Does it give paying/veteran players in-match advantage? (Must be NO)
- [ ] **Session length**: Does it extend average match time beyond mode target? (7/10/14 rounds)
- [ ] **Comeback**: Can a last-place player still win after this feature resolves?
- [ ] **Share moment**: Does this create a screenshot/clip-worthy moment?
- [ ] **Mobile UX**: Is this usable on a 5" phone screen with one thumb?
- [ ] **Anti-pattern check**: Does this match any item on the Hard No-Go list?
- [ ] **Economy impact**: Does this inflate/deflate the GD economy by >15%?
- [ ] **Spectator value**: Is this interesting to watch, not just play?

## Additional Resources

### Reference Files

For detailed analysis and specific guidance, consult:
- **`references/virality-patterns.md`** — Share mechanics, social hooks, spectator design, FOMO events, referral systems
- **`references/anti-patterns.md`** — 12 categories of game-killing mistakes with case studies and solutions
- **`references/similar-games-analysis.md`** — Per-game breakdown of 15 titles with adoptable mechanics and retention loops
- **`references/gaming-frameworks.md`** — Colyseus, Godot 4.x, PostgreSQL, Redis patterns with code-level guidance and scaling strategies
