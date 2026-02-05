# NO GOs: Anti-Patterns That Kill Multiplayer Games

**Research compiled for IMPERIO INMOBILIARIO** — a turn-based, competitive, mobile-first multiplayer real estate tycoon game.

---

## Table of Contents

1. [Pay-to-Win](#1-pay-to-win)
2. [Snowball / Runaway Leader](#2-snowball--runaway-leader)
3. [Player Elimination](#3-player-elimination)
4. [Analysis Paralysis](#4-analysis-paralysis)
5. [Kingmaking](#5-kingmaking)
6. [Information Overload](#6-information-overload)
7. [Toxic Communication](#7-toxic-communication)
8. [Session Length Mismatch](#8-session-length-mismatch)
9. [Unfair RNG](#9-unfair-rng)
10. [Onboarding Cliff](#10-onboarding-cliff)
11. [Dead States](#11-dead-states)
12. [Reward Timing](#12-reward-timing)

---

## 1. Pay-to-Win

### What It Is

Any system where spending real money grants competitive advantage over non-paying players. This includes stat boosts, power-granting loot boxes, level advantages, or premium-exclusive items that affect gameplay outcomes.

### Why It Kills Games

- **Destroys competitive integrity.** Rankings become leaderboards of spending, not skill.
- **Alienates the free-to-play majority.** F2P players form the backbone of matchmaking queues and community. Without them, queue times spike, the community shrinks, and the game hollows out.
- **Creates a trickle-down death spiral.** Casual competitive players burn out facing insurmountable stat gaps. As they leave, queues stretch longer for remaining players. The paying whales eventually have no one to play against.
- **Erodes trust permanently.** Once a game earns a P2W reputation, recovering is nearly impossible.

### Case Studies

**Diablo Immortal (2022)**
- Legendary crests (purchasable) were the only reliable source of 5-star legendary gems. F2P players could only obtain an equivalent gem after approximately 10 years of active play. Full character gear was estimated at $110,000.
- Metacritic user score: 0.2/10 on PC.
- A YouTuber (Jtisallbusiness) spent $100,000 and could no longer find PvP opponents — the matchmaking system couldn't pair him with anyone near his power level. He literally paid to lose access to the game.
- The backlash was so severe that Diablo 4's developers had to publicly commit to no P2W mechanics before the game even launched.

**Star Wars Battlefront 2 (2017)**
- Loot boxes dispensed "Star Cards" granting statistical combat advantages. Premium purchasable power in a $60 retail game.
- Unlocking iconic characters (Darth Vader, Luke Skywalker) required enormous in-game credit grinds — or cash.
- EA's Reddit defense became the most downvoted comment in Reddit history.
- EA removed all microtransactions 24 hours before launch — but the damage was done.
- EA lost approximately $3.1 billion in shareholder value (~10% of market cap).
- Sales tracked at 7 million units vs. an expected 14 million.
- Belgium investigated and classified loot boxes as gambling; legislation followed in multiple countries.
- DICE design director Dennis Brannvall later said: "Not a week goes by without us thinking, 'Imagine if we hadn't launched with loot boxes the way we did.'"

**Clash Royale — Card Levels**
- Card level disparity is the core P2W friction. Upgrading cards requires exponentially more duplicates and gold at higher levels. Paying players reach max level years before F2P.
- Matchmaking uses trophy count, not card levels — so level 10 players routinely face maxed-out opponents.
- The community has extensively proposed card level caps per league, but Supercell has not implemented them, as one player noted it "would ruin Supercell shareholder value since it's an indirect nerf to credit card cycle."
- The debate splits the community: proponents say "skill still matters," opponents counter that a maxed Fireball literally kills troops that an underleveled one leaves alive — no skill compensates for that.

### How to Monetize Without P2W

**Cosmetic-Only Model (Fortnite)**
- Skins, emotes, dances, gliders — zero competitive advantage.
- Battle Pass system: pay once per season, earn cosmetics through play.
- Generated billions while maintaining strict competitive parity.
- Seasonal FOMO drives purchases without granting power.

**Supporter Packs (Path of Exile)**
- All gameplay content free. All microtransactions purely cosmetic.
- Players buy "supporter packs" because they want to support the game, not because they have to.
- Developer Chris Wilson: "We've purposefully divorced any game mechanics from the monetization."
- Sustained a major ARPG for 10+ years.

**Non-Expiring Battle Pass (Halo Infinite)**
- Battle passes never expire — players progress at their own pace.
- Eliminates FOMO pressure while still monetizing cosmetics.

**The Key Principles:**
- Paying should never grant competitive advantage.
- Paying should feel like supporting or personalizing, not investing for power.
- F2P players should be able to reach the exact same competitive ceiling.
- If you sell convenience (faster progression), the ceiling must be the same — only the time to reach it differs.

### Relevance to IMPERIO INMOBILIARIO

The spec correctly defines the Graf Dollar as an in-game currency with no real-money purchase path for competitive advantage. The game should monetize through cosmetic dynasty crests, property visual themes, headline styles, Watcher emotes, and seasonal passes that grant visual prestige. Never sell GD, extra actions, or event protection for real money.

---

## 2. Snowball / Runaway Leader

### What It Is

A positive feedback loop where the player in the lead gains disproportionate advantages, making their lead increasingly insurmountable. The rich get richer; the poor get poorer.

### Why It Kills Games

- **Non-leaders lose motivation.** When the outcome is decided by round 4 of a 10-round game, 6 rounds of play become meaningless theater.
- **Leaders get bored.** Even the winner stops caring when victory feels inevitable.
- **Multiplayer collapses.** In a group game, if 8 of 12 players feel their actions are pointless, 8 people are having a bad time.
- **Replay value vanishes.** Games with runaway leaders feel deterministic. "Why bother playing if early luck decides everything?"

### The Monopoly Disaster (Case Study)

Monopoly is the textbook example of broken game design:

- **Pure positive feedback loop:** More money lets you buy more properties, which charge more rent, which gives you more money. The reward for being ahead is getting further ahead.
- **Exponential compounding:** Building houses/hotels doesn't just add — it multiplies rent. One unlucky landing can bankrupt a struggling player.
- **No catch-up mechanics.** Chance/Community Chest cards are equally likely to be positive or negative regardless of position. The losing player gets no help.
- **Endless dragging.** Despite the outcome being decided early, games drag on for hours. The "killer combo" of runaway leader + distant endgame is the worst possible combination.
- **Player elimination in a long game.** Getting knocked out at minute 45 of a 4-hour game means 3+ hours of watching.
- BoardGameGeek rating: 4.4/10 — one of the lowest-rated "popular" games.
- Monopoly's flaws are so well-documented that it serves as the primary negative example in virtually every game design textbook.

### Proven Solutions

**1. Systematic Leader Friction (Power Grid)**
- Turn order is determined by who is closest to winning — and reverses for critical phases.
- The leader bids first for power plants (disadvantage: no new plants revealed that round).
- The leader buys fuel last (when it's most expensive).
- The leader expands on the map last (opponents can block routes).
- Result: Being in first place is actively disadvantageous in several dimensions. Players strategically manipulate their position, creating deep gameplay. Games are remarkably close until the final round.

**2. Player-Driven Targeted Balancing (Catan's Robber)**
- Rolling 7 lets the roller move the robber to any hex, blocking resource production and stealing from adjacent players.
- Social pressure naturally directs the robber toward the leader.
- Trade denial works similarly — players refuse to trade with whoever's winning.
- Downside: Relies on player knowledge. New players who don't bash the leader can unintentionally kingmake.

**3. Hidden Information**
- If players can't tell who's winning, they can't dogpile the leader — but they also can't exploit snowball advantages.
- Partially hidden scores (Puerto Rico, 7 Wonders) keep outcomes uncertain until endgame scoring.

**4. Negative Feedback Loops (Leader Headwind)**
- In Dominion, victory point cards clog your deck — the more you buy, the less efficient your hands become.
- The leader pays an increasing tax for being ahead, making scoring progressively harder.
- This creates the feeling of a close game even when leads are mathematically significant.

**5. Alternative Victory Conditions**
- Civilization series: military, scientific, cultural, diplomatic victories. A trailing player can pivot to a different path.
- Prevents the leader from controlling every win condition simultaneously.

### Relevance to IMPERIO INMOBILIARIO

The spec already mandates catch-up mechanics, leader friction tax, and fairness caps on events. The formula `OpsCost = floor((NumProperties + TotalUpgrades) / 4)` creates natural overhead scaling. Events target by RiskScore (correlated with wealth/properties/fame), creating inherent leader friction. The key is to ensure these mechanics are strong enough that the round-7 leader in a 10-round game is genuinely uncertain.

---

## 3. Player Elimination

### What It Is

Removing players from the game before it ends. They sit and watch while others play.

### Why It Kills Games

- **Forced inactivity is the cardinal sin.** A player who came to play is now watching. On mobile, they'll simply close the app and never return.
- **It punishes the wrong behavior.** Players who took risks (which makes the game interesting) are eliminated first. The game rewards passivity.
- **Endgame deteriorates.** Fewer players means fewer interactions, fewer options, simpler decisions. The game gets worse as it progresses.
- **Forced cruelty.** In social settings, eliminating a friend feels terrible. The game forces players to be antagonistic in ways that damage relationships.
- **Proportional injustice.** Getting eliminated in round 3 of a 45-minute game means paying 40 minutes of "watching tax."

### Case Studies

**Risk**
- The defining example of elimination gone wrong. Games last 2-4 hours. Players can be eliminated in the first hour. The eliminated player watches friends play for hours with nothing to do.
- Modern variants (Risk Legacy, Risk 2210 AD) have added fixed round limits and objectives to address this.

**Where Elimination Works (and Why)**
- **Love Letter:** Rounds last 2-3 minutes. Eliminated from a round, not the game. Back in action almost immediately.
- **King of Tokyo:** Games last 15-20 minutes. Short enough that elimination stings briefly.
- **Coup:** 5-10 minute games. Elimination is the entire point, and you're back next round fast.
- The pattern: elimination is tolerable only when the total game time is short (under 15 minutes) AND the elimination provides agency (your death was your fault, not random).

### Modern Design Philosophy

The Eurogame tradition explicitly rejects player elimination as a design principle. Modern competitive games keep all players engaged until the end through:

- **Score-based victory** rather than last-man-standing.
- **Degraded positions** instead of elimination — you're weaker but still playing.
- **Rubber-band mechanics** that give trailing players more powerful options.
- **Fixed round counts** that guarantee everyone plays the whole game.

### Relevance to IMPERIO INMOBILIARIO

The spec uses a fixed round count (7-14 rounds depending on mode) with no elimination. Forced sales when cash < 0 degrade a player's position but never remove them. A player who loses all properties still participates in bidding, actions, voting, and deals. This is correct. The game should also ensure that even a heavily degraded player can still influence the outcome and have interesting choices (see Dead States, section 11).

---

## 4. Analysis Paralysis

### What It Is

When a player has so many options, so much information to process, or such high-stakes decisions that they freeze — taking far too long to act, stalling the game for everyone.

### Why It Kills Games (Especially on Mobile)

- **One slow player holds everyone hostage.** In a 12-player game, if one player takes 3 minutes per turn, that's 36 minutes of waiting per round for other players.
- **Mobile context is brutally unforgiving.** Players are on buses, in waiting rooms, during breaks. A 30-second delay feels eternal; a 3-minute one is app-closing territory.
- **Decision fatigue compounds.** By round 5, a player facing 15 options each with 4 sub-options has made hundreds of micro-decisions. Cognitive exhaustion turns strategy into frustration.
- **It correlates with complexity, not depth.** Complex games aren't necessarily deeper. The best games have simple decisions with deep consequences.

### Case Study: Terraforming Mars

- 208 project cards, each with unique effects, costs, prerequisites, and synergies.
- Each round: evaluate 4-10 new cards for purchase, then decide which of your hand to play, in what order, with what resources.
- Drafting variant improves strategy but makes the problem worse — now you're evaluating cards for both yourself and opponents.
- Expansions (Turmoil, Prelude) add more variables to calculate each turn.
- Digital version: a strict timer kicks players who exceed their time limit — punishing AP-prone players by ejecting them from 2-hour games.
- Community consensus: "Terraforming Mars is built like a Yugo but drives like a Ferrari" — the core game is excellent despite AP issues.

### Solutions

**1. Limit Options Per Turn**
- Rather than choosing from 20 actions, present 2-5 meaningful choices.
- IMPERIO's "one Major + one Minor action per round" is exactly right.

**2. Use Timers (But Design Them Well)**
- Hard timers work when all players agree and the game is designed around them.
- Soft timers (declining rewards) work better: a creative approach gives 5 action points if you act within 3 minutes, 3 AP if within 5 minutes, 2 AP after 5 minutes.
- Default action on timeout (no bid = no bid, no action = Pass) eliminates hostage situations.

**3. Break Decisions Into Phases**
- Don't ask "do everything" in one turn. Split into discrete phases: first you bid, then income is calculated, then you pick one action.
- IMPERIO's 11-phase state machine does this perfectly — each phase asks one type of question.

**4. Simultaneous Play**
- 7 Wonders: all players choose and reveal simultaneously. Zero wait time.
- IMPERIO's auction bidding is simultaneous (hidden bids), which is correct.

**5. Ramp Complexity Gradually**
- First rounds should have fewer options. As players build portfolios, decisions naturally increase — but by then players have internalized the system.

**6. Reduce Fear of Regret**
- If individual decisions are devastating (one wrong move = loss), players agonize.
- If individual decisions have moderate impact but long-term patterns matter more, players act faster.
- IMPERIO's "small mistake tolerance" should be explicit: no single bid or action should be game-ending.

### Relevance to IMPERIO INMOBILIARIO

The spec's phase-based turn structure, simultaneous bidding, limited action slots (one Major + one Minor), and server-enforced timers with default actions (no bid = skip, no action = Pass) directly counter AP. The key risk is the DEALS phase — if players can browse complex deal templates with multiple negotiation steps, this phase needs the tightest timer and simplest UI.

---

## 5. Kingmaking

### What It Is

When a player who cannot win determines which of the remaining contenders does win — not through skillful play, but through arbitrary or personal decisions. The "spoiler" or "kingmaker."

### Why It Kills Games

- **The winner didn't earn it.** When Player C decides the game by gifting their resources to Player A instead of Player B, neither Player A nor Player B can claim a legitimate victory.
- **Out-of-game factors leak in.** The kingmaker often decides based on friendship, grudges, revenge for earlier rounds, or metagame alliances — not game state.
- **Breaks the magic circle.** The game stops being about the game and starts being about who Player C likes more. This corrupts the entire competitive premise.
- **Feels worse than losing.** Player B, who played better all game, watches helplessly as Player C hands the win to their friend. No amount of skill could have prevented this.

### Where It's Worst: Diplomatic/Trading Games

Games with unrestricted trading and free-form negotiation are inherently vulnerable:
- A trailing player can dump all resources to a friend.
- A player who can't win can extort others ("give me X or I'll give everything to your rival").
- Diplomacy (the board game) is the quintessential example: a weak player must join a strong alliance or be eliminated, and that choice alone can determine the winner.

### Prevention Strategies

**1. Template-Only Deals (No Free-Form)**
- Restrict trades to predefined templates with balanced terms.
- Prevents resource-dumping, extortion, and "gift everything to my friend."
- IMPERIO's 6 template-only deal types are exactly right.

**2. Hidden/Partially Hidden Scores**
- If players can't clearly see who's winning, they can't intentionally kingmake.
- Puerto Rico hides VP totals. 7 Wonders reveals scores only at the end.
- Partial visibility (show wealth but hide some bonuses) creates uncertainty.

**3. Incentivize All Positions**
- If 2nd, 3rd, 4th place have value (achievements, leaderboard points, seasonal rewards), the "kingmaker" has incentive to play for their own best finish rather than spoiling.
- IMPERIO should award Graf Dollars, XP, or league points for position, not just for winning.

**4. Ensure Positive-Sum Moves Always Exist**
- A player must always be able to make a move that benefits themselves.
- If every action available helps an opponent more than yourself, you're forced into kingmaker territory.

**5. Limit Late-Game Influence of Trailing Players**
- Restrict the impact that low-wealth players can have on leaders.
- Example: a player with 0 properties shouldn't be able to single-handedly determine auction outcomes for top-tier properties.

**6. Rule Penalties for Unsportsmanlike Conduct**
- In tournament settings, obvious kingmaking can be penalized.
- In casual play, social pressure usually suffices.

### A Counter-Argument

Not everyone views kingmaking as purely negative. Designer Cole Wehrle (Root, Pax Pamir) argues that in political/diplomatic games, kingmaking is thematic — managing social capital is part of the game. The winner is the player who played well AND didn't antagonize others into revenge.

But this only works in games explicitly designed around negotiation and political maneuvering. In an economic tycoon game, kingmaking feels arbitrary and unsatisfying.

### Relevance to IMPERIO INMOBILIARIO

Template-only deals (6 types, no free-form) are the primary defense. Additionally: the deal system should not allow one-sided transfers. Every deal must have a cost and a benefit for both parties. The game should also incentivize placement beyond 1st (seasonal XP, league points, achievements for positions 1-3 at minimum).

---

## 6. Information Overload

### What It Is

Presenting too much data on screen simultaneously, overwhelming the player's cognitive capacity and degrading both decision quality and enjoyment.

### Why It Kills Mobile Games Specifically

- **Screen real estate is tiny.** A 6-inch phone screen cannot display what a 27-inch monitor can.
- **Attention is fragmented.** Mobile players are in noisy environments, multitasking, glancing at their phone between other activities.
- **Working memory is limited.** Research shows humans can process approximately 3-5 items simultaneously when learning. New players hit this limit fast.
- **Clutter kills comprehension.** When everything is visible, nothing stands out. Critical information gets lost in the noise.

### The Clean UI Benchmark: Clash Royale

Clash Royale succeeds by showing very little:
- Your 4 cards in hand. Opponent's arena. Elixir bar. Timer. That's it.
- No stats, no spreadsheets, no inventory management during battle.
- Complex card interactions are learned through play, not reading.
- The UI fits perfectly on a phone because it was designed for phones first.

### The Complex UI Trap

Counter-examples include games like Star Citizen (described as "a beautiful, ambitious mess of interfaces") and many 4X strategy games that dump dozens of resource types, production chains, diplomatic statuses, and military units onto a single screen.

### Progressive Disclosure: The Solution

Progressive disclosure reveals information in layers:

**Level 1: Always Visible**
- Only the most critical, moment-relevant information.
- In a bidding phase: the property being auctioned, your cash, the timer.

**Level 2: One Tap Away**
- Context-relevant details available on demand.
- Tap a property to see its full stats, district bonuses, upgrade history.

**Level 3: Drill-Down**
- Deep information for analytical players who want it.
- Portfolio summaries, historical rent trends, opponent property lists.
- Never forced on anyone; always optional.

### Practical Techniques

- **Dynamic HUD elements.** Don't show a health bar when health is full. Show income only during income phase. Show bid UI only during bid phase. Phase-gate the interface.
- **Minimize popups.** Notifications should be unobtrusive and contextual. A banner that fades is better than a modal that blocks.
- **Clear icons with tooltips.** Use visually distinct iconography. Show details on hover/long-press, not by default.
- **Animated hints, not text walls.** Mobile games like Plants vs. Zombies 2 and Pudding Monsters teach mechanics through animated hands and contextual prompts, not instruction manuals.
- **Inflate important values.** Make the "right" option visually distinct. Color-code profitable vs. unprofitable actions.

### Relevance to IMPERIO INMOBILIARIO

The spec's phase-gated state machine naturally limits information per screen — during BID phase, only auction data matters. During ACTION phase, only action choices matter. The Visual Ledger provides a focused 4-second animation of income flow. Key risks:

- The property map with 25 properties must not overwhelm. Use district grouping, highlight only relevant properties (owned, adjacent, affordable).
- The Deal Tray must be non-blocking and minimally intrusive.
- Opponent portfolio data should be one-tap-away, never permanently displayed.
- The Portada (press front page) is inherently information-rich — it should use visual hierarchy aggressively (big headline, smaller sub-stories).

---

## 7. Toxic Communication

### What It Is

Free-form text or voice chat in competitive games that enables harassment, hate speech, griefing through words, and sarcastic abuse.

### Why It Fails in Competitive Games

- **Competition + anonymity + communication = toxicity.** This is nearly a law of multiplayer game design.
- **Moderation costs are enormous.** Real-time AI moderation requires sophisticated NLP, multilingual support, evolving slang detection, and constant calibration between "trash talk between friends" and genuine harassment.
- **False positives alienate players.** Overly strict filters flag innocent conversations and punish competitive banter.
- **Toxicity drives players away.** Research shows failing to stop toxicity in real time leads to lower engagement and lost revenue.
- **Even "safe" systems get abused.** Rocket League's quick chat — designed to enable coordination ("I got it," "Take the shot") — became a toxicity vector. "What a save!" after an opponent's mistake is universally recognized as sarcastic abuse.

### The Spectrum of Solutions

**Full Free Chat (Most Dangerous)**
- Allows maximum expression but maximum toxicity.
- Requires expensive AI moderation, reporting systems, ban infrastructure.
- Even with all this, determined trolls find workarounds.

**Quick Chat / Preset Emotes (Best Balance)**
- Hearthstone: 6 preset emotes only ("Well Played," "Thank You," "Greetings"). No free text between opponents. Even then, sarcastic emote spam led to adding a "Squelch" (mute) button.
- Rocket League: Contextual quick chat phrases. Effective for coordination but still abused sarcastically. Community has proposed limiting quick chat to once per 5 seconds.
- Fortnite: Emotes and dances as expression. No direct text to opponents in-game.

**No Communication at All (Safest but Lonely)**
- Some puzzle/casual games remove all player interaction beyond gameplay moves.
- Eliminates toxicity but also eliminates social connection — which is a key retention driver.

### Recommended Approach for Competitive Mobile Games

1. **Quick Chat with curated phrases.** 8-15 preset phrases in the game's language.
2. **Rate-limit messages.** Maximum 1 message per 10 seconds prevents spam.
3. **Mute option.** One-tap mute per player. Muted state persists for the match.
4. **No free-text chat between opponents.** Period.
5. **Party/team chat only for friends** who have mutually opted in.
6. **Report function** with delayed consequences (chat ban escalation: 24h -> 72h -> 30 days).

### Relevance to IMPERIO INMOBILIARIO

The spec already specifies `data/quickchat.json` with 12 preset Spanish phrases. This is exactly right. The game should:

- Never allow free-text chat between opponents.
- Rate-limit quick chat to prevent spam.
- Allow one-tap mute per player.
- Consider removing quick chat during high-stakes phases (BID, DEALS) to prevent pressure/harassment.
- Keep Watcher chat separate and optionally hideable by owners.

---

## 8. Session Length Mismatch

### What It Is

When a game's match duration doesn't fit the platform, context, or audience's available attention.

### Why It Kills Mobile Games

- **Average mobile session: 6-13 minutes.** The median is 6 minutes; the average for top performers is 13 minutes. Strategy games can stretch to 50+ minutes, but those are outliers.
- **Attention spans on mobile are brutal.** Players context-switch constantly — notifications, conversations, arriving at their bus stop. A game that demands 30+ minutes of uninterrupted attention on mobile is fighting human nature.
- **Abandonment cascades.** When one player leaves a multiplayer game, the experience degrades for everyone. Long sessions increase abandonment probability per player, and the effect compounds with player count.
- **Natural break points matter.** Players can play a match "before the meeting starts" or "while waiting for food." If the match doesn't fit that window, they don't start it at all.

### Case Studies

**Monopoly: Too Long for Its Own Good**
- Average game: 1-3 hours. Some games last 4+.
- No natural stopping point. No fixed round limit. No mercy rule.
- Players frequently house-rule "1 hour limit" or "3 trips around the board" — because the designer didn't.
- The game's excessive length is inseparable from its other problems (runaway leader, elimination).

**Fall Guys: Right-Sized**
- Full show: 5-6 rounds, approximately 15 minutes total.
- Individual rounds: 60-90 seconds.
- Fits mobile attention windows. Quick enough to play "just one more."
- Failure doesn't sting because you're back in a new game within seconds.

**Clash Royale: Perfect Match Duration**
- Matches: 3-5 minutes.
- Overtime: 1-3 minutes maximum.
- Total maximum: ~7 minutes per match. Fits any mobile context.
- Win or lose, you can immediately play another.

### Design Principles

1. **Matches must fit natural mobile breaks.** 10-20 minutes maximum for strategy games. 3-7 minutes for action games.
2. **Quick mode must exist.** Even if a standard game is 30 minutes, offer a 10-minute quick variant.
3. **Design for interruption.** Save state, allow reconnection (60s grace period), enable bot substitution for disconnects.
4. **Natural break points within matches.** End-of-round summaries, phase transitions, and press headlines serve as natural pauses.
5. **Make matches complete stories.** Even a 10-minute match should have a beginning (early bids), middle (portfolio building), and climax (final events and scoring).

### The Mobile Benchmark

| Genre | Target Session | Notes |
|-------|---------------|-------|
| Casual/Puzzle | 3-5 min | Quick in/out, high sessions/day |
| Card/Strategy | 10-15 min | Can tolerate longer if deeply engaged |
| Multiplayer competitive | 7-15 min | Sweet spot for mobile competitive |
| Complex strategy | 20-30 min max | Above this, abandonment rates spike |

### Relevance to IMPERIO INMOBILIARIO

The spec offers three modes: Quick (7 rounds), Standard (10 rounds), Long (14 rounds). With server-enforced phase timers, Quick mode should target 12-18 minutes, Standard 20-30, Long 35-45. Quick mode is the mobile priority. The timers must be aggressive enough to keep pace — if BID is 30s, ACTION is 25s, DEALS is 30s, VISUAL_LEDGER is 4s, and other phases are 10-15s, a full round takes approximately 2-3 minutes. 7 rounds = ~15-20 minutes for Quick mode. This fits.

---

## 9. Unfair RNG

### What It Is

Randomness that feels capricious, punitive, or uncontrollable — where outcomes appear arbitrary rather than exciting.

### The Perception Problem

Players don't hate randomness. They hate feeling powerless. The same random event can feel thrilling or infuriating depending on:

- **Can I mitigate it?** (Buy insurance, diversify, prepare)
- **Does it scale with my choices?** (High risk-taking = high event chance)
- **Is there a pattern I can learn?** (Not pure chaos)
- **Am I ever guaranteed a floor?** (Bad luck protection)

### When RNG Feels Unfair

- **High impact, zero agency.** A random event destroys your best property and there was nothing you could have done to prevent, mitigate, or insure against it.
- **Streaks without protection.** Three negative events in a row targeting the same player with no bad luck prevention. Mathematically possible; experientially devastating.
- **Hidden mechanics.** Players who don't understand the probability model assume the worst. Transparency (or the illusion of it) is critical.
- **Winner-determined by dice.** If the final outcome is decided by a coin flip, the preceding 30 minutes of strategy felt pointless.

### Solutions

**1. Pseudo-Random Distribution (Dota 2)**
- A "25% chance" doesn't use true randomness. The actual probability increases after each failure and resets after success.
- Two triggers in a row become near-impossible. Six failures in a row become near-impossible. The average converges to 25%.
- Players perceive this as "fair 25%" even though it's actually biased — biased toward fairness.

**2. Pity Timers / Bad Luck Protection**
- After N failed rolls, guarantee a success.
- Clash Royale's chest cycle: deterministic 240-chest sequence guaranteeing specific chest types at fixed intervals. Feels random but is actually guaranteed.
- Genshin Impact: 5-star character guaranteed within 90 pulls. Probability increases sharply after 75 pulls ("soft pity").

**3. Input Randomness vs. Output Randomness**
- **Input randomness (good):** The property available for auction is random — but your bid decision is deterministic. You respond to randomness with skill.
- **Output randomness (bad):** You make a decision and then dice determine if it works. Your skill was irrelevant to the outcome.
- Best practice: randomness sets the stage; player choice determines the outcome.

**4. Fairness Caps**
- Limit how many negative events can hit one player per game.
- Limit how much total damage events can do to a single player.
- IMPERIO's fairness caps (spec Part 8) are exactly this.

**5. Transparency and Signaling**
- Show players what drives their event probability (RiskScore, properties, fame).
- If players understand why they're targeted, it feels like consequence, not caprice.
- Market Signals visible each round let players anticipate trends.

**6. Agency Over Exposure**
- Insurance, Compliance Audits, and Security Contracts let players reduce their risk.
- Players who get hit hard feel it's their fault for not preparing — which is psychologically far better than feeling victimized.

### Relevance to IMPERIO INMOBILIARIO

The spec's event system has strong RNG design:
- Events target based on RiskScore (player agency over exposure).
- Insurance and Security Contracts provide mitigation.
- Fairness caps limit stacking.
- RiskScore is partially transparent (players can see their risk factors).

Additional recommendations:
- Implement pseudo-random distribution for event targeting. Don't hit the same player on consecutive rounds unless no other valid target exists.
- Show players their RiskScore and its components after each round.
- Ensure the worst-case scenario for any single event never exceeds ~25% of a player's wealth.

---

## 10. Onboarding Cliff

### What It Is

The moment when a new player's guidance vanishes and they're dropped into full complexity without support. The "cliff" between tutorial and real game.

### The Numbers

- ~20% of players are lost during first-time user experience in successful F2P games. For less successful games, the number is far worse.
- A game has approximately 15 minutes to show its core loop AND convince the player to keep playing.
- Working memory during learning: ~3 items maximum. New tasks have much higher cognitive load than familiar ones. Overloading working memory doesn't create challenge — it creates frustration and confusion.

### Common Anti-Patterns

**1. The Info Dump**
- 10 screens of text before the player does anything.
- "Here are the 11 phases, 52 events, 6 deal types, 3 election candidates..."
- Players retain almost nothing. They close the app.

**2. The Tutorial That Doesn't Teach the Real Game**
- Tutorial shows simplified mechanics. Real game has completely different complexity.
- Player feels competent in tutorial, then incompetent in real game. Trust is broken.

**3. The Cliff: Tutorial -> Abyss**
- Great tutorial, then zero guidance. Player is suddenly alone with full complexity.
- No contextual hints, no progressive revelation, no "Help" button.

**4. The Endless Tutorial**
- "The real game starts at level 10." Players give up at level 3.
- Dragging out onboarding kills the player's curiosity and patience.

**5. The Forced Tutorial**
- No skip option. Experienced players (or returning players) must sit through basics.
- Sizable player segment hates tutorials. Forcing them hurts retention.

### Case Study: Among Us (Success)

- Core loop is radically simple: do tasks OR find the impostor.
- "How to Play" section exists but is optional — players click it voluntarily because they want to understand.
- One round teaches more than any tutorial could. Learning through play.
- Game continuously refines onboarding based on player feedback.
- Result: massive adoption across age groups and gaming experience levels.

### Case Study: Chess.com (Progressive Complexity)

- Low rule complexity: 6 piece types, simple movement rules. A beginner can play legal moves within 5 minutes.
- High analytical complexity: mastery requires thousands of hours. But you can start playing (badly) immediately.
- Chess.com layers complexity: lessons, puzzles, rated games, analysis tools — each introduced when the player is ready.
- Rating system ensures beginners play beginners. You're never dropped into the deep end.

### Best Practices

1. **Teach through play, not text.** The first match should BE the tutorial. Contextual prompts during actual gameplay.
2. **Introduce one concept per round.** Round 1: bidding. Round 2: actions. Round 3: events. Round 4: deals.
3. **Make the core loop learnable in 60 seconds.** IMPERIO targets "Fast to learn (60 seconds)" — this means the first match should need only: "Bid on a property. Pick an action. See what happens."
4. **Allow tutorial opt-out.** Always. Some players learn by doing.
5. **Continue hints beyond the tutorial.** First 3 matches should have optional contextual tips that fade after the player demonstrates understanding.
6. **Make players feel smart.** Early decisions should have an obviously good choice. Reward the player for "figuring it out" even when the answer was telegraphed.
7. **Never punish exploration.** First-match mistakes should have minimal consequences. Bot-filled first match with reduced stakes.

### Relevance to IMPERIO INMOBILIARIO

The spec targets "Fast to learn (60s)" — one Bid + one Action per round. This is the correct core loop. The onboarding risk is the surrounding complexity: events, deals, moods, elections, insurance, compliance, fame, dynasties. All of this should be invisible to new players and progressively revealed:

- Match 1: Bid + Action only. Events happen but are explained inline. No deals.
- Match 2: Deals introduced. Events explained more briefly.
- Match 3: Full game. Optional tips available.
- Bot tutorial mode (spec A8) should use this progressive approach.

---

## 11. Dead States

### What It Is

A game position where a player technically remains in the game but has no meaningful choices. They cannot win, cannot meaningfully influence outcomes, and cannot voluntarily exit — they're just going through the motions until the game ends.

### Why It's Worse Than Elimination

At least eliminated players can do something else. Dead-state players are trapped in a game where their actions don't matter but they're still required to take turns. This is the worst of both worlds.

As one designer put it: "Player elimination hasn't truly been removed from many games. You can very much be eliminated as early as the first move. What's missing is the dignity of actually being kicked out and allowed to do something else. Instead you just trudge along with zero chance of doing anything."

### Monopoly's Mortgage Spiral (Case Study)

- Player lands on a hotel and can't pay rent.
- They mortgage properties to pay. Mortgaged properties generate no rent.
- Next round, with no income and fewer assets, they land on another property and mortgage more.
- The spiral accelerates: fewer properties -> less income -> more mortgages -> fewer properties.
- The player is technically "in the game" for 5-10 more rounds while being slowly drained.
- Every decision during this spiral is meaningless — they're choosing which finger to cut off, not making strategic choices.
- This is not gameplay. This is paperwork for a predetermined conclusion.

### Detection and Resolution

**Detecting Dead States:**
- Track wealth trajectory over rounds. If a player's wealth has declined for 3+ consecutive rounds and is below the median by more than 50%, they may be in a dead state.
- Track meaningful decision count. If a player has no affordable bid, no affordable action, and no viable deal partner, their decision space has collapsed.
- Track win probability. If statistical analysis shows a player's chance of winning has dropped below 5%, they're effectively dead.

**Resolving Dead States Gracefully:**

**1. Emergency Injection**
- Give struggling players a small cash injection or free property when they hit a threshold.
- Frame it narratively: "A mysterious investor has taken interest in your dynasty" or "Government subsidy for struggling landlords."

**2. High-Risk, High-Reward Options**
- Offer trailing players access to risky actions that leading players don't have.
- "Double-or-nothing" bids. "Desperate gamble" actions with asymmetric payoffs.
- These keep trailing players engaged because they have a meaningful (if unlikely) path back.

**3. Pivot Objectives**
- When winning is impossible, give players alternative goals: "Highest single-round income," "Most improved position," "Most events survived."
- Achievements and per-match XP that reward interesting play regardless of final position.

**4. Voluntary Spectator Mode**
- If a player recognizes they're dead, let them voluntarily switch to Watcher mode.
- They keep their match XP, gain Watcher-specific engagement options (betting, voting on moods).
- This preserves their agency: they chose to pivot, rather than being trapped.

**5. Mercy Acceleration**
- When the game detects a dead state for multiple players, slightly accelerate the remaining rounds (shorter timers, fewer phases).
- The game ends faster, putting everyone out of their misery without dramatically changing the competitive dynamic for leaders.

### Relevance to IMPERIO INMOBILIARIO

The spec's forced-sale mechanism when cash < 0 prevents the most extreme dead states, but a player who has sold all properties and has minimal cash is still in a dead state. Recommendations:

- Implement "Subsidy" mechanic: if a player's wealth drops below 25% of the median for 2 consecutive rounds, they receive a small GD injection (framed as a "government grant" or "investor interest").
- Track and surface a "Comeback Score" — even trailing players should see metrics that show improvement.
- Ensure at least one affordable property exists in each auction for the poorest player (via base price distribution in the property pool).
- Award meaningful post-match XP for engagement (actions taken, deals proposed, votes cast) not just for winning.

---

## 12. Reward Timing

### What It Is

The schedule, frequency, and perceived fairness of rewards — too sparse creates boredom, too frequent creates numbness, and poorly timed rewards create obligation instead of excitement.

### The Psychology

- **Dopamine is triggered by anticipation, not just receipt.** The brain's reward system activates more strongly from the expectation of a reward than from receiving it. This is why variable rewards are more compelling than fixed ones.
- **Variable Ratio Reinforcement (VRR)** is the most powerful engagement driver. Rewards given after an unpredictable number of actions create the strongest behavioral reinforcement. This is the slot machine principle — and it's why 60%+ of top mobile games use some form of VRR.
- **Fixed rewards create predictability.** Fixed ratio schedules (every 5th win = reward) create a sense of reliability but lower excitement. Best used for baseline progression.
- **The combination is key.** Layer fixed progression (predictable baseline) with variable bonuses (exciting spikes) for optimal engagement.

### Clash Royale's Chest System: The Hybrid Model

Clash Royale's system is deceptively clever — it appears random but is actually deterministic:

- **Fixed 240-chest cycle** that repeats infinitely. Every win earns the next chest in sequence. The cycle guarantees: 180 Silver, 52 Golden, 4 Giant, 4 Magical chests per cycle.
- **Contents within chests are variable.** The chest type is fixed; the cards inside are random. This creates the illusion of randomness while guaranteeing progression.
- **Crown Chest resets daily.** Every 10 crowns unlocks a bonus chest. Creates a daily engagement target.
- **Result:** Players feel the excitement of variable rewards (what cards will I get?) while actually receiving guaranteed progression (you will get a Magical Chest every ~60 wins, no matter what).

### When Rewards Become Chores

Academic research ("Daily Quests or Daily Pests?", ACM CHI PLAY 2022) studied 178 players and found:

- Players who engaged naturally with rewards reported higher intrinsic motivation and healthier play patterns.
- Players who played primarily to collect rewards reported higher external motivation, amotivation, and obsessive play patterns.
- Some described daily rewards as "a chore" or "like a job."
- FOMO (Fear of Missing Out) on daily rewards became a "guilt lever" — players logged in not because they wanted to play but because they feared losing streak bonuses.
- One community member called daily login rewards "nicotine for video games."

### When Retention Mechanics Backfire

- **Mandatory daily logins feel like taxes.** If missing one day breaks a streak or locks content, the game transforms from entertainment into obligation.
- **Over-scheduling creates burnout.** When every day has mandatory quests, weekly challenges, seasonal events, and limited-time offers, players feel overwhelmed.
- **FOMO as a design tool erodes trust.** Countdown timers and "LAST CHANCE!" banners work short-term but breed resentment long-term. Players eventually feel manipulated and quit entirely — the exact opposite of the intended effect.
- **Reward quality degrades.** When everything gives a reward, rewards lose meaning. If opening a chest is exciting on day 1 but tedious on day 100, the system has failed.

### Healthy Reward Design Principles

**1. Rewards Within Matches (Intrinsic)**
- Every round should have a dopamine moment: winning an auction, seeing income flow in, surviving an event, pulling off a deal.
- The Visual Ledger animation (4s of watching money flow) is a reward itself — visual feedback of progress.
- Press headlines are rewards — seeing your dynasty named is recognition.

**2. Post-Match Rewards (Extrinsic)**
- Match completion should always feel rewarding regardless of placement.
- XP, league points, and achievements should reward participation AND performance.
- Variable bonuses for notable achievements ("Comeback King," "Event Survivor," "Deal Mogul").

**3. Session Rewards (Not Daily Obligations)**
- Replace daily login rewards with session-based rewards: "Play 3 matches today" instead of "Log in every day."
- Allow catch-up: if you miss Monday, Tuesday's reward includes Monday's. No punishment for having a life.

**4. Seasonal Rewards (Long-Term)**
- Seasonal league progression with meaningful tiers.
- Seasonal decay prevents eternal grinding — your rating resets partially each season, creating fresh competition.
- End-of-season rewards based on peak rank, not current rank (prevents anxiety about losing rank before season ends).

**5. Anti-Chore Design**
- Never require players to do things they don't enjoy for rewards.
- If a "quest" says "Lose 5 matches," that's a chore. If it says "Win an auction for a property in Miraflores," that's interesting.
- Optional bonuses, never mandatory gates.

### Relevance to IMPERIO INMOBILIARIO

The spec's in-match reward cadence is strong: every round has visible income, events, press headlines, and Visual Ledger animations. Post-match: the Portada (front page), achievements, and dynasty progression provide satisfying closure.

Key risks to avoid:
- Don't implement daily login rewards that punish absence.
- Don't create FOMO with time-limited content that expires.
- Do implement session-based rewards ("Play 3 matches this week").
- Do use the Graf Dollar metagame economy for long-term progression without P2W.
- Do make every match feel complete and rewarding, regardless of placement.

---

## Summary Matrix: Anti-Patterns vs. IMPERIO INMOBILIARIO Defenses

| Anti-Pattern | Game-Killing Effect | IMPERIO's Built-In Defense | Remaining Risk |
|---|---|---|---|
| **Pay-to-Win** | Kills competitive integrity | No real-money purchase of GD or power | Must resist temptation to add P2W later |
| **Snowball** | Non-leaders stop caring | OpsCost scaling, leader tax, fairness caps, events target by RiskScore | Verify catch-up math: round-7 leader should win <60% of the time |
| **Elimination** | Players leave the app | Fixed round count, forced sales not elimination | Ensure degraded players still have interesting choices |
| **Analysis Paralysis** | One player holds everyone hostage | Phase-gated turns, limited actions (1 Major + 1 Minor), server timers, default actions on timeout | DEALS phase needs tight timer and simple UI |
| **Kingmaking** | Winner didn't earn it | Template-only deals, no free-form transfers | Must ensure deals are balanced (cost + benefit both sides) |
| **Info Overload** | New players overwhelmed | Phase-gated UI, Visual Ledger focused animation | Property map with 25 properties needs careful grouping |
| **Toxic Chat** | Players harassed, community poisoned | Quick chat with 12 preset Spanish phrases | Need rate-limiting and mute |
| **Session Length** | Doesn't fit mobile context | 3 modes (Quick/Standard/Long) with enforced timers | Quick mode must stay under 20 minutes |
| **Unfair RNG** | Players feel victimized | Events target by RiskScore, insurance exists, fairness caps | Implement pseudo-random distribution, no same-player consecutive targeting |
| **Onboarding Cliff** | Lose 20%+ of new players | "Fast to learn (60s)" design goal | Must progressively reveal complexity across first 3 matches |
| **Dead States** | Trapped in pointless game | Forced sales prevent zeroing out | Need subsidy mechanic and alternative goals for trailing players |
| **Reward Timing** | Game feels like a chore | In-match Visual Ledger, headlines, achievements | Avoid daily login obligations and FOMO mechanics |

---

## Sources

### Pay-to-Win
- [Washington Post — Diablo Immortal Pay-to-Win](https://www.washingtonpost.com/video-games/2022/06/14/diablo-immortal-pay-to-win-monetization/)
- [GameSpot — Diablo Immortal Player Can't Find Matches](https://www.gamespot.com/articles/diablo-immortal-player-paid-to-win-too-much-can-no-longer-find-pvp-matches/1100-6506060/)
- [ExpertBeacon — Is Diablo Immortal Still Pay-to-Win?](https://expertbeacon.com/is-diablo-immortal-still-pay-to-win-1/)
- [FandomWire — Star Wars Battlefront 2 Lootbox Controversy](https://fandomwire.com/star-wars-battlefront-2-lootbox-controversy-explained/)
- [CNBC — EA Removes Paid Loot Boxes](https://www.cnbc.com/2018/03/16/ea-vows-to-never-offer-paid-loot-boxes-in-its-controversial-star-wars-battlefront-ii-game.html)
- [Screen Rant — Battlefront 2 Was DICE's Lowest Moment](https://screenrant.com/star-wars-battlefront-2-loot-box-controversy-rock-bottom/)
- [ZLeague — Is Clash Royale Pay-to-Win?](https://www.zleague.gg/theportal/clash-royale-paywin/)
- [ZLeague — Clash Royale P2W Dilemma](https://www.zleague.gg/theportal/clash-royale-the-pay-to-win-dilemma-a-free-to-play-players-frustration/)
- [Meegle — Game Monetization For Competitive Games](https://www.meegle.com/en_us/topics/game-monetization/game-monetization-for-competitive-games)
- [Gamedeveloper — Path of Exile Ethics of F2P](https://www.gamedeveloper.com/business/the-mechanics-and-ethics-of-free-to-play-in-i-path-of-exile-i-)

### Snowball / Runaway Leader
- [Oakleaf Games — Runaway Leader, Rubber Banding, and Feedback](https://oakleafgames.wordpress.com/2014/02/13/game-theory-runaway-leader-rubber-banding-and-feedback/)
- [Envato Tuts+ — The Snowball Effect and How to Avoid It](https://code.tutsplus.com/the-snowball-effect-and-how-to-avoid-it-in-game-design--cms-21892a)
- [Medium — Catch Me If You Can: Catch-Up Mechanics](https://fantastic-factories.medium.com/catch-me-if-you-can-the-runaway-leader-and-catch-up-mechanics-53f0356c440d)
- [The Thoughtful Gamer — Catch-Up Mechanisms](https://thethoughtfulgamer.com/2017/03/28/catch-up-mechanisms/)
- [BoardGameGeek — Compendium of Solutions](https://boardgamegeek.com/geeklist/204332/compendium-solutions-rich-gets-richer-runaway-lead)
- [BuzzFeed — Why Monopoly Is The Worst Game](https://www.buzzfeed.com/tomchivers/monopoly-sucks)
- [Medium — Intentional Broken Mechanic in Monopoly](https://medium.com/design-bootcamp/a-ux-study-the-intentional-broken-board-game-mechanic-in-monopoly-b512f5622bfb)
- [Brandon The Game Dev — 7 Lessons from Monopoly](https://brandonthegamedev.com/7-lessons-from-monopoly-for-aspiring-board-game-designers/)

### Player Elimination
- [League of Gamemakers — Game Elements: Elimination](https://www.leagueofgamemakers.com/game-elements-elimination/)
- [Player Lair — Why Player Elimination Has Become Taboo](https://playerlair.net/why-player-elimination-has-become-taboo-in-board-games/)
- [There Will Be Games — Slow Death: Player Elimination](https://therewillbe.games/articles-essays/9111-slow-death-player-elimination-in-board-games)
- [Pine Island Games — Player Elimination Mechanics](https://www.pineislandgames.com/blog/player-elimination-mechanics-clank)

### Analysis Paralysis
- [Medium — Analysis Paralysis: Smart Game Design](https://medium.com/theuglymonster/analysis-paralysis-how-smart-game-design-can-keep-everyone-happy-6e97f2e72b10)
- [League of Gamemakers — Designing to Prevent AP, Part 1](https://www.leagueofgamemakers.com/designing-games-to-prevent-analysis-paralysis-part-1/)
- [League of Gamemakers — Designing to Prevent AP, Part 2](https://www.leagueofgamemakers.com/designing-games-to-prevent-analysis-paralysis-part-2/)
- [Gamers.Online — Addressing Analysis Paralysis](https://gamers.online/blog/addressing-analysis-paralysis-in-board-games/)

### Kingmaking
- [Skeleton Code Machine — Is Kingmaking a Problem to Be Solved?](https://www.skeletoncodemachine.com/p/kingmaking)
- [Wikipedia — Kingmaker Scenario](https://en.wikipedia.org/wiki/Kingmaker_scenario)
- [DiVA Portal — Mitigating Kingmaking in Multiplayer Board Games (PDF)](http://www.diva-portal.org/smash/get/diva2:1876522/FULLTEXT01.pdf)
- [No Rerolls — Kingmaking in Board Games](https://norerolls.co.uk/2021/03/23/kingmaking-in-board-games/)
- [GDC Vault — King Me: A Defense of Kingmaking](https://www.gdcvault.com/play/1025683/Board-Game-Design-Day-King)

### Information Overload
- [Wayline — 17 Ways to Minimize Cognitive Load in Game UI](https://www.wayline.io/blog/minimize-cognitive-load-game-ui)
- [UXPin — Progressive Disclosure](https://www.uxpin.com/studio/blog/what-is-progressive-disclosure/)
- [UX Planet — Progressive Disclosure for Mobile Apps](https://uxplanet.org/design-patterns-progressive-disclosure-for-mobile-apps-f41001a293ba)
- [JustInMind — Game UI Design Principles](https://www.justinmind.com/ui-design/game)

### Toxic Communication
- [Confluent — Real-Time Toxicity Detection in Games](https://www.confluent.io/blog/confluent-databricks-detecting-gaming-toxicity/)
- [Dignitas — Rocket League Community and Toxicity](https://dignitas.gg/articles/rocket-league-the-community-and-how-to-avoid-toxicity)
- [GamersRdy — Understanding Toxicity in Rocket League](https://gamersrdy.com/blog/2021/08/28/understanding-toxicity-in-rocket-league/)
- [The Global Gaming — How What A Save Became Popular](https://theglobalgaming.com/rocket-league/what-a-save)

### Session Length
- [Udonis — Mobile Game Session Length](https://www.blog.udonis.co/mobile-marketing/mobile-games/session-length)
- [GameRefinery — Session-Length Restriction](https://www.gamerefinery.com/3-things-to-know-about-session-length-restriction-when-designing-a-free2play-game/)
- [Gamedeveloper — How First Session Length Impacts Performance](https://www.gamedeveloper.com/business/how-first-session-length-impacts-game-performance)
- [Mobile Free To Play — Mobile Session Design](https://mobilefreetoplay.com/mobile-session-design/)

### Unfair RNG
- [Medium — Modeling Fair and Fun Randomness via Bad Luck Protection](https://medium.com/@niklasvmoers/designing-fair-and-fun-randomness-in-video-games-via-bad-luck-protection-48f2c2262cfa)
- [GameDev.net — Luck in Games: Why RNG Isn't the Answer](https://www.gamedev.net/tutorials/game-design/game-design-and-theory/luck-in-games-why-rng-isnt-the-answer-r3877/)
- [COGconnected — Why Randomness Keeps Us Hooked](https://cogconnected.com/2025/08/why-randomness-keeps-us-hooked-the-role-of-rng-in-game-design/)
- [Time to Loot — RNG vs. Player Agency](https://www.timetoloot.com/gaming/rng-vs-player-agency/)

### Onboarding
- [Inworld — Game UX Best Practices for Onboarding](https://inworld.ai/blog/game-ux-best-practices-for-video-game-onboarding)
- [Game Wisdom — The Importance of Onboarding](https://game-wisdom.com/critical/onboarding-game-design)
- [Gamedeveloper — How Onboarding Should Be Applied to Tutorials](https://www.gamedeveloper.com/design/how-onboarding-should-be-applied-to-tutorials)
- [The Acagamic — 5 Proven Game Onboarding Techniques](https://acagamic.com/newsletter/2023/04/04/dont-spook-the-newbies-unveiling-5-proven-game-onboarding-techniques/)
- [Celia Hodent — The Gamer's Brain: UX of Onboarding](https://celiahodent.com/gamers-brain-ux-onboarding/)

### Dead States
- [Decision Space Podcast — The Decision Space of Monopoly](https://www.decisionspacepodcast.com/articles/the-decision-space-of-the-game-monopoly-by-jim-dambrosia)
- [University XP — Meaningful Choices](https://www.universityxp.com/blog/2019/8/6/meaningful-choices)
- [Game Wisdom — Hierarchy of Fail States](https://game-wisdom.com/critical/hierarchy-of-fail-states-game-design)
- [The Almighty Guru — Unwinnable State](https://www.thealmightyguru.com/Wiki/index.php?title=Unwinnable_state)

### Reward Timing
- [ACM — Daily Quests or Daily Pests? (Academic Paper)](https://dl.acm.org/doi/10.1145/3549489)
- [Gamedeveloper — Reward Schedules and When to Use Them](https://www.gamedeveloper.com/business/reward-schedules-and-when-to-use-them)
- [Number Analytics — Ultimate Guide to Reward Schedules](https://www.numberanalytics.com/blog/ultimate-guide-reward-schedules-game-design)
- [Gamedeveloper — The Science of Daily Rewards](https://www.gamedeveloper.com/business/the-science-craft-of-designing-daily-rewards----and-why-ftp-games-need-them)
- [ClashDecks — Chest Mechanics Guide](https://clashdecks.com/guides/beginner/chest-mechanics-guide)
- [Ultimate Gacha — Daily Login Fatigue](https://ultimategacha.com/daily-login-fatigue-why-gacha-games-respect-player-time/)
