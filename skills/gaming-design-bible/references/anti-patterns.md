# Game Design Anti-Patterns ("No-Go" Bible)

Comprehensive catalog of design mistakes that kill multiplayer games. Each anti-pattern includes real-world failure cases, root cause analysis, and proven solutions.

---

## 1. Pay-to-Win (P2W)

### What It Is
Any mechanic where spending real money provides in-match competitive advantage over non-paying players.

### Why It Kills Games
- Destroys competitive integrity — matches feel predetermined by wallet size
- Triggers negative reviews, refund waves, and media backlash
- Whales leave when non-payers leave (nobody to beat)
- Regulatory scrutiny increasing globally (Belgium, Netherlands loot box bans)

### Case Studies

**Diablo Immortal (2022):** Fully maxing a character required ~$110,000 USD. Metacritic user score: 0.2/10. Despite $100M+ revenue, destroyed Blizzard's reputation. Key lesson: even massive IP can't survive P2W backlash.

**Clash Royale Card Levels:** Cards have levels that increase stats. Higher-level cards dominate lower-level ones regardless of skill. Created a "frustration funnel" pushing F2P players toward spending. While financially successful, it created persistent community resentment and "P2W" label that limited growth.

**Star Wars Battlefront II (2017):** Loot boxes provided gameplay advantages. Launch backlash so severe that EA removed the system entirely. Cost estimated at $3.1B in stock value. Led to legislative investigations.

### Solution
- **Cosmetic-only monetization**: Skins, emotes, badges, card backs, board themes
- **Equal match starts**: Every player begins with identical resources regardless of persistent wallet
- **Prestige-only persistent currency**: GD/trophies show achievement history, never buy power
- **Battle pass model**: Seasonal content tracks that reward playtime, not spending

### How This Applies to IMPERIO INMOBILIARIO
GD is prestige-only. Match starts are always equal (500K/650K/800K by mode). No persistent item provides in-match advantage. Cosmetics are the monetization path.

---

## 2. Snowball / Runaway Leader

### What It Is
When gaining an early advantage creates a compounding lead that's impossible to overcome. The rich get richer.

### Why It Kills Games
- 5 of 6 players have no hope of winning by mid-game → they mentally check out or quit
- Reduces strategic depth (early game is all that matters)
- Creates "I already know who won" feeling by Round 3

### Case Studies

**Monopoly:** Classic snowball. First player to get a color set snowballs via houses/hotels. Other players slowly bleed out over 2+ hours. Average Monopoly game: ~90 minutes of knowing you'll lose. This is THE cautionary tale.

**Risk:** Armies compound. The player who captures the most territories early gets the most reinforcements. Continental bonuses accelerate the snowball. Combined with player elimination = terrible experience.

**Catan (partially):** Ore + wheat engine can snowball via development cards + cities. Mitigated somewhat by robber targeting the leader, but still problematic with experienced vs. novice players.

### Solutions

| Mechanic | How It Works | Game Reference |
|----------|-------------|----------------|
| **Leader Friction Tax** | Wealth leader pays extra OpsCost each round | Power Grid turn order |
| **Fairness Caps** | Maximum cash damage per Owner per round (GD 150K cap) | Mario Kart blue shell |
| **Event Targeting** | Higher RiskScore for wealthier players | Catan's robber |
| **Consolation Mechanics** | R1 subsidy (GD 50K) if no properties after R1 | Modern euro games |
| **Diminishing Returns** | OpsCost scales with portfolio size + upgrades | Power Grid plant maintenance |
| **Turn Order Advantage** | Leader bids last (loses information advantage) | Power Grid |

### How This Applies to IMPERIO INMOBILIARIO
- OpsCost = `floor((NumProperties + TotalUpgrades) / 4) × 8,000` — scales with success
- LeaderFriction: top quartile pays GD 8,000; #1 pays GD 15,000 if Wealth > 1.2× #2
- FairnessCap: max GD 150,000 cash damage per Owner per round
- HardHitSpacing: max 1 event with cash loss ≥ GD 100,000 per 2-round window
- R1 Consolation: GD 50,000 if you own 0 properties at end of R1

---

## 3. Player Elimination

### What It Is
Removing players from the game before it ends. Once eliminated, they sit and watch (or more likely, leave).

### Why It Kills Games (Especially Mobile)
- Eliminated player had a bad experience → negative review, uninstall
- Waiting for friends to finish is pure frustration
- On mobile, eliminated player opens another app and never comes back
- Multiplayer games need ALL players engaged until the end for healthy ecosystem

### Case Studies

**Monopoly:** Bankruptcy eliminates players. A 4-player game can eliminate someone in 45 minutes, with 90+ minutes remaining. That player's evening is ruined.

**Risk:** Military elimination. A player can be knocked out in 30 minutes of a 3-hour game.

**Poker tournaments:** Early bust-outs are the norm. Works because of buy-back options and cash game alternatives. Does NOT work in casual mobile contexts.

### Solutions
- **Forced sale at 80%** instead of bankruptcy — keeps player in game with cash
- **Debt system with cap** — players can go negative but only to GD 750,000 debt
- **"Consolation" income** — players with 0 properties still earn minimum income
- **Spectator-with-power role** — eliminated players become Watchers who can vote on Moods, affecting remaining players
- **Short match length** — Even worst-case, the game ends in 7-14 rounds (10-25 minutes)

### How This Applies to IMPERIO INMOBILIARIO
- No player elimination exists. Forced sales at 80% keep players liquid.
- Debt cap of GD 750,000 prevents infinite spiral.
- Match length capped at 7/10/14 rounds.
- Even 0-property players participate in voting, deals, and can recover via loans + auctions.

---

## 4. Analysis Paralysis

### What It Is
When players have too many options, too much information, or insufficient time pressure, causing turns to drag.

### Why It Kills Games
- Other players wait → boredom → frustration → quit
- On mobile, waiting = opening another app
- Multiplayer games live and die by pace

### Case Studies

**Terraforming Mars:** Late-game turns with 15+ cards in hand, multiple resource types, and combo possibilities can take 5+ minutes per player. 4-player games regularly hit 3+ hours.

**Civilization (board game):** Each turn involves military, economy, technology, and diplomacy. Information overload + option overload = glacial pace.

### Solutions

| Solution | Implementation | Game Reference |
|----------|---------------|----------------|
| **Phase timers** | 15-30s per phase, server proceeds regardless | Chess clock |
| **Major/Minor action split** | Reduce ACTION phase to 2 choices (1 major + 1 minor) | Wingspan |
| **Default action** | If timer expires, "Pass" is selected automatically | Digital board games |
| **Progressive information** | Show relevant info only during current phase | Clash Royale |
| **Smart suggestions** | Server generates pre-filled deal/action suggestions | Modern digital games |

### How This Applies to IMPERIO INMOBILIARIO
- Every phase has a timer (15-30s). Server proceeds regardless of client response.
- ACTION phase: pick 1 Major + 0-1 Minor (not "choose from everything")
- Smart Deal Suggestions reduce deal-creation friction
- Visual Ledger shows only current-round deltas (not full history)

---

## 5. Kingmaking

### What It Is
When a player who cannot win determines who does win, usually through spite or favoritism. The losing player's final actions decide the outcome for others.

### Why It Kills Games
- Winner feels unearned victory
- Loser of the king-made decision feels cheated
- Creates social conflict in friend groups
- Repeated kingmaking makes players avoid the game

### Case Studies

**Catan:** Player in last place trades exclusively with their friend, handing them the win. Nothing prevents this in the rules.

**Diplomacy:** The entire endgame is kingmaking. Losing players choose which alliance to honor, determining the winner.

**Munchkin:** Players collectively decide when to help/hinder, creating social dynamics that override strategy.

### Solutions
- **Template-only deals** — No free-form trading. All deals use predefined templates with server-validated fairness
- **Anti-spite mechanics** — Deals must be "mutually beneficial" (both parties gain value)
- **Anonymized late-game actions** — In final rounds, reduce information about who benefits from deals
- **Server-validated trades** — Reject clearly imbalanced deals (GD 1 for a GD 1M property)

### How This Applies to IMPERIO INMOBILIARIO
- 7 deal templates only (no free-form). Server validates each deal type.
- Non-compete agreements have enforcement (Romper Pacto costs both action slots)
- Events are server-targeted (no player chooses who gets hit)
- Auction system is open bid (no targeted gifting of properties)

---

## 6. Information Overload

### What It Is
Showing too much data simultaneously, especially on mobile screens. Players can't process what matters.

### Why It Kills Games
- New players overwhelmed → quit in tutorial
- Even experienced players make suboptimal decisions when overloaded
- Mobile screens have ~1/4 the real estate of desktop
- Cognitive load reduces enjoyment (play should feel intuitive, not like work)

### Solutions

| Principle | Implementation |
|-----------|---------------|
| **Progressive disclosure** | Show only current-phase info. Hide irrelevant data. |
| **Visual hierarchy** | Cash balance = large + prominent. Debt details = expandable panel. |
| **Animation as teaching** | Visual Ledger teaches economy by showing cash flow |
| **Tooltip depth** | Tap for summary, long-press for detail |
| **Color coding** | Green = good, red = bad, gold = premium. Universal. |
| **Maximum 5 items** | No list should show more than 5 items without scrolling |

### Reference: Clash Royale's UI Mastery
- Match screen shows: your towers, their towers, elixir bar, hand of 4 cards
- That's it. No stats overlay, no damage numbers flying, no minimap
- All complexity is BENEATH the surface, revealed only when you seek it
- Result: 5-year-olds can play, pros find infinite depth

---

## 7. Toxic Communication

### What It Is
Free-text chat in competitive games leads to harassment, hate speech, and toxicity that drives players away.

### Why It Kills Games
- Toxic players drive away 10x more players than they retain
- Moderation costs are enormous (human + AI review)
- Legal liability in many jurisdictions
- Children play these games — duty of care
- One toxic interaction can cause permanent churn

### Case Studies

**League of Legends:** Spent $100M+ on toxicity systems (Tribunal, behavioral systems, chat restrictions). Toxicity still their #1 player complaint. Riot's own research: toxic games cause 5x higher churn.

**Rocket League:** Replaced free chat with Quick Chat presets. Toxicity dropped dramatically while maintaining communication. "What a save!" became iconic.

**Hearthstone:** Emote-only communication. Six preset emotes. Clean, expressive, non-toxic. "Well played" is universally understood.

### Solutions
- **Quick Chat presets** — 12 Spanish phrases covering all needed communication
- **Rate limiting** — 2 messages per phase per Owner (prevents spam)
- **No user-generated text** — No free-text input in any game context
- **Speech bubble display** — Fades after 3 seconds (no persistent chat log to screenshot for harassment)
- **Mute option** — Per-player mute for even Quick Chat messages

### How This Applies to IMPERIO INMOBILIARIO
- 12 Quick Chat presets in Spanish (from QC-01 to QC-12)
- Rate limit: 2 messages per phase per Owner
- No free-text chat. No custom messages.
- Deal text uses template phrases only
- Headlines are server-generated (no player-authored content)

---

## 8. Session Length Mismatch

### What It Is
When match duration doesn't fit the platform's attention span. Mobile games over 20 minutes lose players. Desktop games under 5 minutes feel unsatisfying.

### Why It Kills Games
- **Too long (mobile):** Player gets interrupted (call, notification, bus stop), abandons mid-match, feels guilty, avoids game
- **Too short:** No time for strategic depth, feels like a coin flip
- **Unpredictable length:** Player can't commit because they don't know the time commitment

### Case Studies

**Monopoly (too long):** Average game: 90+ minutes. Most house-rule versions: 2+ hours. Players quit mid-game constantly. The most famous board game in history is also the most abandoned mid-session.

**Fall Guys (right):** 15-minute shows. Perfect for "one more round" psychology. Short enough for a bus ride, long enough for investment.

**Clash Royale (right):** 3-minute matches (5 with overtime). Can play anywhere. "I have 3 minutes" = "I can play a match."

### Solutions
- **Fixed round count** — Not "play until someone wins" but "7/10/14 rounds, highest Wealth wins"
- **Display time estimate** — "Quick Mode: ~10 min" shown before match start
- **No overtime** — Match ends at defined round, period
- **Save state for reconnection** — If player disconnects, 60s to rejoin (bot autopilots)

### How This Applies to IMPERIO INMOBILIARIO
- Quick Mode: 7 rounds (~10 min)
- Standard Mode: 10 rounds (~18 min)
- Long Mode: 14 rounds (~25 min)
- Time displayed upfront. No overtime. Fixed round count.

---

## 9. Unfair RNG

### What It Is
When randomness feels arbitrary and uncontrollable. Players who lose to "bad luck" blame the game and quit.

### Why It Kills Games
- "I played perfectly and still lost" = uninstall
- Repeated bad RNG creates learned helplessness
- New players who lose to RNG in first match never return
- Perception of fairness matters more than actual fairness

### Solutions

| Mechanic | How It Works |
|----------|-------------|
| **Insurance system** | Players can buy protection against bad events (choice = agency) |
| **Fairness caps** | Max damage per round prevents catastrophic single-event loss |
| **Hard-hit spacing** | Max 1 major event per 2-round window |
| **Weighted probability** | Events target leaders more (feels fair because they "deserve it") |
| **Mitigation options** | Security contracts, PR Push, insurance — player agency over fate |
| **Visible probability** | RiskScore is transparent — players know their risk level |
| **Pity timer** | After 3 consecutive negative events, suppress the 4th |

### How This Applies to IMPERIO INMOBILIARIO
- Insurance (Basic/Full) provides tiered event mitigation
- FairnessCap: max GD 150,000 damage per Owner per round
- HardHitSpacing: max 1 event ≥ GD 100,000 per 2-round window
- Security contracts protect against specific event types
- RiskScore formula is deterministic and transparent
- MOOD "Calma chicha" suppresses events entirely for a round

---

## 10. Onboarding Cliff

### What It Is
When a game's complexity overwhelms new players in the first 5 minutes, causing them to quit before experiencing the fun.

### Why It Kills Games
- Industry average: 70% of mobile game players quit within first session
- Complex games: up to 90% first-session quit rate
- You never get a second chance at a first impression
- Every feature you show in tutorial is one more reason to quit

### Solutions
- **Learn by playing** — First match is a 4-player game against 3 bots at Easy difficulty
- **Progressive complexity** — R1: just bid. R2: bid + collect rent. R3: bid + rent + first event. Build up.
- **Highlight one thing** — Each round's tutorial overlay highlights ONE new concept
- **Skip option** — Experienced players can skip tutorial entirely
- **Tooltips, not manuals** — Context-sensitive help on tap, not upfront info dump
- **Safe first match** — Tutorial match doesn't count for leaderboard (low stakes)

### Reference: Among Us Onboarding
- No tutorial. Drop player into match.
- Game rules learned through 2-3 plays.
- Total rule set fits on one screen.
- Result: 500M downloads.

---

## 11. Dead States

### What It Is
When a player has no meaningful decisions but the game continues. They're technically playing but have no agency.

### Case Studies

**Monopoly bankruptcy spiral:** Player mortgages everything, lands on hotels, can't pay. Spends 5+ turns just passing Go and paying rent until bankrupt. Zero agency for 15+ minutes.

**Risk low-territory:** Player with 1 territory and 3 armies. Can't attack. Just waits to be eliminated.

### Solutions
- **Forced sale at 80%** — Prevents asset-poor cash starvation
- **Bank loans** — Always available (GD 150,000) to re-enter the economy
- **Consolation income** — R1 subsidy ensures everyone can participate
- **Short match length** — Even worst case, game ends in 7-14 rounds
- **Deal access** — Even cash-poor players can offer deals to get back in
- **Mood/Vote participation** — All players can vote regardless of wealth

---

## 12. Reward Timing

### What It Is
Getting the cadence of rewards wrong — either too frequent (dopamine fatigue, rewards feel meaningless) or too sparse (players feel unrewarded, quit between reward events).

### Optimal Reward Schedule

| Frequency | Reward Type | Dopamine Effect |
|-----------|------------|-----------------|
| **Every round** | Rent income animation | Steady drip — keeps engagement |
| **Every 2-3 rounds** | Event resolution (positive) | Surprise — variable ratio reinforcement |
| **Every match** | Wealth added to persistent GD | Achievement — session completion reward |
| **Every 3-5 matches** | League promotion/progress | Milestone — visible progression |
| **Weekly** | Leaderboard position update | Status — social comparison |
| **Monthly** | Season rewards | Culmination — big payoff |

### The Variable Ratio Principle
The most addictive reward schedule (slot machines, Clash Royale chests) is **variable ratio reinforcement** — rewards come at unpredictable intervals. In IMPERIO INMOBILIARIO:
- Events are random (some help, some hurt — surprise)
- Auction outcomes depend on other players (unpredictable)
- Deal offers are player-driven (social unpredictability)
- Mood effects vary each round (market surprises)

This creates natural variable-ratio engagement without artificial loot boxes.
