# Virality Mechanics Research for IMPERIO INMOBILIARIO

Compiled from analysis of Monopoly GO!, Fall Guys, Among Us, Clash Royale, Brawl Stars, Chess.com, Fortnite, and academic game design research. All recommendations are filtered for a **turn-based multiplayer real estate tycoon** (2-12 players, 7-14 rounds, mobile-first, Lima setting).

---

## 1. Share-Worthy Moments

The single most important virality driver is giving players something they *want* to share without being asked. Share-worthy moments share three traits: **surprise**, **social proof** (look what I did), and **visual clarity** (immediately readable as an image/clip).

### What triggers sharing in reference games

| Game | Share Trigger | Why It Works |
|------|--------------|--------------|
| Monopoly GO! | High-multiplier dice roll landing on a jackpot tile | Risk/reward payoff visible in one frame. The 10x multiplier burn creates "I can't believe that worked" moments. |
| Fall Guys | Winning a crown (especially as last bean standing) | Binary outcome + rare achievement + silly avatar = screenshot-worthy. |
| Clash Royale | Overtime 1HP tower clutch or prediction spell | Dramatic reversal compressed into a 10-second replay. |
| Among Us | Ejecting the wrong person / perfect impostor win | Social betrayal moment that creates a *story* worth retelling. |
| Chess.com | Brilliant move (!!) classification | External validation of skill — the system tells you "that was genius." |
| Brawl Stars | Star Player designation on winning team | Public MVP recognition on the end screen. |

### Actionable mechanics for Imperio Inmobiliario

1. **"Golpe Maestro" Detection**: Server-side detection of dramatic plays. Examples:
   - Winning an auction by exactly GD 1,000 (minimum increment)
   - Buying a property in Round 7 that flips you from last to first place in Wealth
   - Completing a District Set in a single round
   - Surviving a catastrophic event with exactly GD 0 remaining
   - Breaking a pact at the exact moment it would have cost you the game

   When detected, the server tags the moment with a `golpeMaestro` flag. The client renders a special animation and auto-generates a share card with the headline.

2. **"Fortuna o Ruina" Moments**: Any round where a player's Wealth changes by more than 40% in either direction. The press system already generates headlines — attach these to share assets automatically.

3. **Auction Overbid Shame / Snipe Glory**: When someone wins an auction at 3x+ the base price (overpaid), tag it as "Se lo llevaron caro." When someone wins at base price because nobody else bid, tag it as "Robo del siglo."

4. **Event Survival Badge**: When a player is hit by a devastating event (earthquake, scandal) and still ends the round in top 3, auto-generate a "Sobrevivi" share card.

---

## 2. Social Hooks

Social hooks create **obligation loops** — reasons to come back that are tied to other people, not just the game itself.

### How reference games create viral loops

**Monopoly GO!** runs the most aggressive social loop in mobile gaming:
- Contact import on first launch populates your game world with real friends
- Friends appear on your board — you attack/steal from them, they get revenge notifications
- Sticker trading requires friends (you can't complete albums alone)
- Partner Events require 4-player teams, forcing recruitment
- The "Invite Bar" gives tiered dice rewards (30 dice at 5 invites, bigger at 10)
- Daily "what happened while you were away" notifications name specific friends who attacked you

**Clash Royale / Brawl Stars** (Supercell pattern):
- Clans/Clubs with donation mechanics (give cards to clanmates = everyone benefits)
- Clan War participation requires minimum members, forcing recruitment
- Clan chat where replays are shared directly — social proof within the group
- Weekly clan leaderboard resets create recurring competition

**Among Us**:
- Zero-friction "play with friends" via room codes
- The game is fundamentally *about* your friend group — the social dynamic IS the gameplay
- Post-game accusations and stories create organic word-of-mouth

### Actionable mechanics for Imperio Inmobiliario

1. **Dynasty Recruitment Loop**: The Dynasty Ladder (already in spec 3.5.2) is the strongest hook. Make Dynasty rankings aggregate all members' lifetime GD (already in spec 3.5.6). This means recruiting skilled players directly improves your Dynasty rank. Add:
   - **Dynasty Chat**: Persistent between matches. Members share portadas, discuss strategy.
   - **Dynasty Challenges**: Weekly targets ("La Dinastia X debe acumular GD 10M esta semana"). Rewards: Dynasty XP multiplier, exclusive Dynasty cosmetic.
   - **Dynasty Recruitment Board**: Open slots visible to non-members. "Buscamos inversores" message customizable by Dynasty leaders.

2. **"Revancha al toque" with Stakes**: The rematch flow (spec 3.5.2) should include a "double or nothing" option where winners can wager Dynasty XP on the rematch. This turns a one-match session into a best-of-3 spiral.

3. **Friend Invite Deep Link**: When a player shares their referral link, the recipient should land in a lobby where the inviter is waiting — not just the app store. Reduce friction between "click link" and "playing together" to under 90 seconds. The current spec (3.5.4) gives +50 Dynasty XP per referral; consider also giving the referrer a cosmetic border on their portada showing how many people they recruited.

4. **Absent Player Notifications**: "Tu amigo [Name] te quitó una propiedad mientras dormías" — this is the Monopoly GO pattern of naming specific friends in push notifications to create emotional urgency to return.

---

## 3. Session Pacing for Virality

Short, complete experiences are more shareable than long ones. The ideal viral session produces a shareable moment within the time it takes to watch a TikTok compilation.

### Reference game session lengths

| Game | Session Length | Why It Works for Sharing |
|------|--------------|------------------------|
| Among Us | 8-15 min per round | One complete story arc with setup, tension, climax, resolution |
| Fall Guys | 5-12 min per show | Elimination creates natural narrative compression |
| Brawl Stars | 2-3 min per match | Fits in a single TikTok or Instagram Story |
| Monopoly GO! | 2-5 min per session (dice spending) | Continuous micro-sessions with spike moments |
| Clash Royale | 3-5 min per battle | Overtime mechanic guarantees a dramatic final 60 seconds |

### Key insight: TikTok-length content

Short-form video platforms favor content under 90 seconds. Games that produce 15-30 second highlight moments get the most organic sharing. The TikTok algorithm promotes content with >70% completion rate — meaning your shareable clips need to hook in 1-2 seconds and resolve within 30-60 seconds.

AI tools now auto-detect "highlight moments" from streams and produce vertical clips. Games that produce clean, legible UI at mobile resolution are inherently more clippable.

### Actionable mechanics for Imperio Inmobiliario

1. **Quick Mode is the viral mode**: The 7-round Quick match (~20-30 min total) is the right length for a complete play session. But for virality, you need **sub-moments within the match** that are individually shareable:
   - Each auction (BID phase) is a self-contained drama: 40 seconds of tension with a clear winner/loser. This is your "TikTok moment."
   - The Visual Ledger animation (4 seconds, spec Part 5) showing wealth changes is a natural clip boundary.
   - Event resolution (earthquake hits, someone loses a property) is a 10-15 second dramatic beat.

2. **Auto-Highlight Reel**: At match end, auto-compose a 30-second vertical video from the 3 most dramatic moments (largest wealth swings, closest auctions, most impactful events). Render client-side using recorded state snapshots. One-tap share to TikTok/Instagram/WhatsApp.

3. **Phase-by-Phase Micro-Sharing**: Allow players to share individual round results mid-match. After the PRESS phase generates headlines, offer a "Compartir titular" button that creates a branded image with the headline + player stats for that round. This creates multiple share opportunities per match instead of just one at the end.

4. **Round Timer Visibility**: The 40-second bid/action timers create natural tension peaks. Make the final 10 seconds visually dramatic (screen shake, audio cue, timer turns red) — this is when spectators and streamers react, producing organic clips.

---

## 4. Emotional Peaks

Emotional peaks are what separate games people *talk about* from games people just play. The design literature identifies a specific pattern: **tension → uncertainty → resolution → social reaction**. All four must be present for word-of-mouth to activate.

### How reference games manufacture emotional peaks

**Auction tension** (Monopoly GO!, Power Grid, Modern Art):
- The "Dollar Auction" effect — once committed, players escalate past rational limits
- Visible commitment (everyone sees your bid) adds social pressure
- Time pressure (countdown) forces snap decisions
- Winner's curse (overpaying) creates both schadenfreude and regret

**Dramatic reversals** (Clash Royale overtime, Mario Kart blue shell):
- Catch-up mechanics ensure the leader is never safe
- Last-round scoring shifts create "anything can happen" finales
- Hidden information reveals (Among Us impostor vote) flip assumptions

**Betrayal moments** (Among Us ejection, Catan trade backstabs):
- Social contracts are established then broken
- The betrayer gains while the betrayed reacts
- Third-party observers pick sides, amplifying the drama

**Near-miss emotional design**:
- Chess.com's accuracy score (93% = "so close to perfect")
- Losing an auction by GD 1,000 hurts more than losing by GD 100,000
- Getting hit by an event the round AFTER your insurance expired

### Actionable mechanics for Imperio Inmobiliario

1. **"Romper Pacto" as the Betrayal Mechanic**: Already in spec (3.2) — breaking a pact is limited to once per match, costs both Major and Minor action slots. This is your Among Us moment. When it happens:
   - Full-screen dramatic animation
   - Press headline: "TRAICION EN [DISTRITO]: [Player] rompe pacto con [Player]"
   - Both players' portadas at match end reference the betrayal
   - The broken-pact victim gets a "Traicionado" badge they can display

2. **Last-Round Wealth Reveal Delay**: In the final round, hide the exact leaderboard positions until the SNAPSHOT phase. Show only relative indicators ("Vas primero... por ahora"). The final wealth calculation becomes a dramatic reveal — like opening an envelope.

3. **Auction Escalation Design**: The BID phase already has 40-second timers. Add:
   - **Bid History Visibility**: Show each player's bid in real-time (not sealed). This creates social pressure and Dollar Auction dynamics.
   - **"Guerra de Ofertas" Trigger**: When 3+ players bid on the same property and the price exceeds 2x base, trigger a special visual effect and press headline.
   - **Watcher Reactions**: Watchers can react with emoji during auctions (already in spec). These reactions should be visible to bidders, adding crowd pressure.

4. **Catch-Up Emotional Arcs**: The leader friction tax (spec Part 8, fairness caps) is your rubber-banding mechanic. Frame it narratively:
   - Leader gets more events targeting them (higher RiskScore)
   - Press headlines create narratives around the leader: "El imperio de [Player] tambalea"
   - When a trailing player overtakes the leader, trigger: "CAMBIO DE GUARDIA: [Player] destronó a [Leader]"

5. **Near-Miss Feedback**: When a player loses an auction by a small margin, show exactly how close they were: "Perdiste por GD 5,000." When an event hits and insurance would have covered it: "Si hubieras tenido seguro, habrías salvado GD 80,000." These near-misses create emotional spikes and teach through regret.

---

## 5. Portada / End-Screen Share Cards

The post-match share card (Portada) is your single highest-leverage viral asset. Every match produces one. If even 5% of players share it, the organic reach compounds.

### How reference games design share cards

**Chess.com Game Review**:
- Accuracy score (0-100) — single vanity number that begs comparison
- Evaluation graph — visual narrative of who was winning at each point
- Move classifications (Brilliant/Great/Blunder) — memorable moments highlighted
- Opening name — educational hook that makes the player feel knowledgeable
- Performance rating — "you played like a 1800" validation

**Brawl Stars Battle Cards**:
- Customizable pre-match identity card (icons, pins, titles, backgrounds)
- Star Player MVP designation on the end screen
- End screen scoreboard: kills, deaths, damage, healing
- Ranked backgrounds that display progression tier

**League of Legends Wild Rift**:
- Auto-generated banners from match stats
- Personal and team achievement variants
- Designed specifically for social media dimensions

**Clash Royale**:
- Replay sharing via link (not video — re-simulated on viewer's device)
- Clan chat replay sharing for social proof within the group
- Crown count and tower health as primary share metrics

### Best practices (synthesized)

1. **One vanity metric dominates**: Chess.com's accuracy, Brawl Stars' Star Player, Clash Royale's crown count. Pick ONE number that players will compare.

2. **Include a visual narrative**: The Chess.com evaluation graph tells a story. A flat line = boring game, a zigzag = dramatic match. This visual alone makes people want to share.

3. **Brand everything**: Logo, game name, and download link/QR must be on every share card. The card IS an ad.

4. **Two dimensions**: 1200x630 for link previews (Facebook/Twitter) and 1080x1920 for Stories (Instagram/WhatsApp Status/TikTok). Already in spec (3.5.1).

5. **Deep link embedded**: The share card should link back to the game with a referral code baked in. Every share is a potential install.

6. **Design for dark messaging apps**: Most shares go to WhatsApp, iMessage, Messenger — not public feeds. The card must be legible at thumbnail size on dark backgrounds.

### Actionable Portada design for Imperio Inmobiliario

**Data to include on the Portada:**

| Element | Purpose |
|---------|---------|
| Player avatar + Dynasty name | Identity / brand recall |
| Final Wealth (GD amount) | The vanity metric. Big number = bragging rights |
| Placement (1st/2nd/3rd with podium visual) | Clear outcome |
| Wealth graph (7-14 data points) | Visual narrative of the match arc |
| Number of properties owned at end | Portfolio size bragging |
| Most valuable property name + district | Lima flavor |
| Best headline from the match | Narrative hook ("TRAICION EN MIRAFLORES") |
| Match mode + round count | Context |
| Dynasty badge / rank tier | Long-term progression display |
| Game logo + QR code with referral link | Acquisition |

**"Portada" variants to auto-generate:**

| Variant | Trigger | Data Emphasis |
|---------|---------|---------------|
| Victory Portada | Win the match | Gold frame, Wealth + properties, Dynasty rank |
| Comeback Portada | Win after being in bottom 3 for 3+ rounds | Wealth graph with dramatic upswing, "De la ruina al imperio" tagline |
| Betrayal Portada | Used "Romper Pacto" | Both players shown, headline about the betrayal |
| Survivor Portada | Hit by 3+ events and still finished top half | Event icons survived, "Inquebrantable" badge |
| Bankrupt Portada | Went bankrupt (forced out) | Humorous/dramatic, "Caiste pero con estilo", shows peak wealth before crash |
| Perfect Landlord | Never lost a property all match | Properties grid, "Intocable" badge |
| Watcher Favorite | Won "Favorito del publico" vote | Heart icons, watcher count, crowd validation |

**Mid-match share card ("Logro"):**
- Triggered by achievements (first property, first district set, first package)
- 1080x1920 story format
- Shows the specific achievement with current match stats
- Includes "Estoy jugando ahora" CTA with room link (for games in progress)

---

## 6. Spectator Virality

Among Us became the 4th most-watched game on Twitch in 2020 not because of gameplay depth but because it was **entertaining to watch**. The spectator-to-player conversion pipeline is: Watch stream/clip -> understand the game in 30 seconds -> want to try it -> download -> play -> stream/share -> loop.

### What makes games watchable

| Design Element | Why It Works | Reference |
|----------------|-------------|-----------|
| Simple visual language | Viewers understand what's happening without playing | Among Us crewmates, Fall Guys beans |
| Social deduction / hidden info | Viewers know something players don't (or vice versa) | Among Us impostor identity |
| Short rounds (3-15 min) | Drop-in viewers get a complete experience | Brawl Stars, Fall Guys |
| Reaction-worthy moments | Streamer reactions drive engagement | Clutch wins, betrayals, surprise events |
| Spectator tools (stats, free camera) | Dedicated spectator UI adds production value | Esports observers |
| Meme-friendly aesthetics | Simple, distinctive character designs become viral templates | Among Us "sus", Fall Guys costumes |
| Streamer mode | Protects against stream sniping, hides lobby codes | Among Us "Hide Code", Valorant |

### Key insight from research

"Game rounds are generally short and sweet — you can watch for three to five minutes and still have an enjoyable experience. In other games that have 45-60 minute rounds, a three-minute viewing session isn't exactly rewarding."

The three pillars of spectator engagement (from CHI research): **Agency** (can the audience influence the game?), **Pacing** (does the game maintain dramatic rhythm?), **Community** (does watching feel social?).

### Actionable mechanics for Imperio Inmobiliario

1. **Watcher Role IS the Spectator Mode**: The spec already has Watchers (up to 8 per match) with mood votes, rumor buttons, and auction betting. This is inherently Twitch-friendly. But add:
   - **Watcher Commentary Overlay**: Watchers see a simplified UI with key metrics. Allow Watchers to switch between following specific Owners (like camera angles in a broadcast).
   - **Watcher Prediction Market**: Extend auction betting to match outcome betting. "Who wins the match?" predictions at the start of each round. Correct predictions earn Watcher Fame, which unlocks Watcher-specific cosmetics.
   - **Watcher Highlight Cam**: Watchers can "bookmark" moments during the match. At match end, their bookmarks contribute to the auto-highlight reel.

2. **Streamer Mode**: Add a toggle that:
   - Hides the room code from the screen
   - Replaces player names with Dynasty names (for privacy)
   - Adds a slight delay (15-30 seconds) to prevent stream sniping during auctions
   - Shows a "Watching [Streamer]" badge for Watchers who joined via a stream link

3. **"Novela Inmobiliaria" Narrative Layer**: The press system (spec Part 10) already generates headlines. For streamers, overlay these headlines as breaking news tickers during gameplay. This turns every match into a telenovela that's entertaining even when the mechanical gameplay isn't visually dramatic.

4. **Clip-Friendly UI Design**:
   - Ensure all important information is in the center 80% of the screen (safe zone for vertical cropping)
   - Use large, high-contrast numbers for Wealth changes (readable at 480p mobile)
   - Animate wealth changes with dramatic particle effects that are visually striking in clips
   - The Visual Ledger animation (4 seconds) should be designed to look good as a standalone clip

5. **Twitch Integration (Future)**: Allow Twitch chat to participate as Watchers via chat commands. `!bid Player3` to predict auction winner, `!rumor Player5` to use the rumor mechanic via Twitch. This converts passive viewers into active participants.

---

## 7. Referral Mechanics

The best referral systems in mobile gaming share a pattern: **tie the reward to gameplay**, not just to the act of inviting. Players should *need* friends to play optimally, not just get a bonus for inviting them.

### Reference game referral systems

**Monopoly GO! (best-in-class mobile referral)**:
- Contact import on first launch
- Invite Bar with tiered rewards (30 dice at 5 invites, bigger hauls at 10)
- Friends appear on your board — you can attack and steal from them
- Sticker trading requires friends (can't complete albums solo)
- Partner Events require 4-player teams
- Multi-channel invite sharing (SMS, WhatsApp, Instagram Stories)
- Referred friends get a starter bonus, making them more likely to accept

**RAID: Shadow Legends (progression-tied referral)**:
- Rewards unlock based on referred friend's level (not just install)
- Legendary hero unlocks when 3 referred friends reach level 50
- Referral Points from friends' continued play and IAPs
- Creates long-term incentive to keep friends engaged

**World of Warcraft Recruit-A-Friend**:
- Unique mounts and pets only available through referrals
- XP bonuses when playing with your recruit (cooperative play incentive)
- Rewards tied to referred friend's subscription duration

**HQ Trivia (gameplay-tied referral)**:
- Inviting friends gave extra lives (which had real monetary value via IAP)
- The reward was directly tied to core gameplay need

### Best practices (synthesized)

1. **Trigger share at emotional peaks**, not in menus. After a win, after a dramatic moment — not in settings.
2. **Reward the referrer AND the referee**. Double-sided incentives convert better.
3. **Tie rewards to referred friend's progression**, not just install. This ensures quality referrals.
4. **Use tiered milestones** with visible progress bars. "3/5 friends invited" with a progress bar is more motivating than "invite friends."
5. **Create gameplay dependency** on having friends. If friends make you stronger/richer/more competitive, the invitation is self-motivated.
6. **Deep link directly into the game**, not the app store. Reduce steps between clicking and playing together.
7. **Limited-time referral bonuses** create urgency.

### Actionable mechanics for Imperio Inmobiliario

1. **Upgrade the existing referral system** (spec 3.5.4):
   - Current: +50 Dynasty XP per referred player who completes Tutorial
   - Proposed additions:
     - **Tiered Bar**: Progress bar showing referral milestones (3, 5, 10 invites)
     - **Milestone 3**: Exclusive portada frame ("Reclutador")
     - **Milestone 5**: Exclusive avatar accessory + 200 Dynasty XP
     - **Milestone 10**: Exclusive Dynasty title + permanent +10% Dynasty XP bonus
     - **Ongoing**: +5 Dynasty XP every time your referred friend completes a match (up to 50 matches)

2. **Dynasty Recruitment as Referral Mechanic**: Frame referrals as "recruiting for your Dynasty." The Dynasty leaderboard (spec 3.5.6) aggregates all members' GD. More members = more aggregate wealth = higher Dynasty rank. This makes referral inherently competitive — you're not just inviting friends, you're building an empire.

3. **"Juega Conmigo" Deep Link**: One-tap invite that:
   - Opens a pre-filled WhatsApp/SMS message with game link
   - Links directly to a lobby where the inviter is waiting
   - Auto-adds both players as friends in-game
   - Gives referee a starter bonus (extra GD 50,000 in their first match)

4. **Post-Match Invite Prompt**: After a win (emotional peak), show a "Invita a tu pata" prompt. After a loss, show "Necesitas refuerzos?" Both trigger the invite flow. Never show invite prompts during neutral moments.

---

## 8. FOMO & Limited Events

FOMO mechanics convert anxiety about missing out into engagement. Statistics show games leveraging FOMO see a 23% increase in retention during promotional periods, and players in seasonal events show 40% higher retention than non-participants.

### How reference games create FOMO

**Fortnite Battle Pass**:
- Seasonal cosmetics available only for a fixed duration
- Once the season ends, items are permanently inaccessible
- Tiered progress with daily/weekly challenges
- Creates a "calendarized obligation" to play regularly

**Genshin Impact Limited Banners**:
- Rare characters available for short periods (2-3 weeks)
- Characters may not return for 6+ months
- Players save resources and plan logins around banner schedules
- Gacha + time scarcity = powerful FOMO

**Monopoly GO! Events**:
- Rotating events (Golden Blitz, Tycoon Racers, Partner Events)
- Each event has its own reward track with exclusive stickers
- Events last 1-3 days, creating urgency
- Community Chest events require collective participation
- Event completion often requires social coordination

**Pokemon GO Community Day**:
- 3-hour window for exclusive Pokemon spawns
- Exclusive moves only available during the event
- Social meetup component (real-world gathering)
- Recurring monthly pattern creates anticipation

### Key FOMO mechanisms

1. **Countdown timers**: Visual urgency cues that activate loss aversion
2. **Exclusive rewards**: Items unobtainable after the event ends
3. **Social visibility**: Friends can see your exclusive items, creating envy
4. **Tiered progress**: Season passes with levels you must grind before time runs out
5. **Social pressure**: Clan/Dynasty events where your absence hurts the team

### Ethical balance

Overusing FOMO leads to player fatigue and backlash. The best approach: mix time-limited and permanent content. Always offer a path to earn exclusive items later (at higher cost or lower priority), so missing an event feels bad but not catastrophic.

### Actionable mechanics for Imperio Inmobiliario

1. **Weekly Dynasty War** (time-limited event):
   - Every weekend (Friday 6PM - Sunday 11PM Lima time)
   - Dynasties compete for aggregate GD earned during the event window
   - Top 3 Dynasties earn exclusive Dynasty cosmetics (banner colors, portada borders)
   - Cosmetics rotate every month — miss a month, miss that design forever
   - Absent Dynasty members hurt the team's aggregate, creating social pressure to participate

2. **"Ronda Especial" Limited Modes** (rotating every 2 weeks):
   - **Solo Terremoto**: All 52 events can fire every round (chaos mode)
   - **Duelo de Magnates**: 1v1 format, 5 rounds, winner-take-all
   - **Subasta Ciega**: All bids are sealed (hidden from other players)
   - Each mode has its own leaderboard and exclusive portada variant
   - Available only for 2 weeks, then rotates

3. **Seasonal Themes** (quarterly):
   - **Q1 - Verano Limeno**: Beach/costa properties get +1 rent bonus, exclusive summer portada backgrounds
   - **Q2 - Fiestas Patrias**: Peruvian independence themed, special patriotic Dynasty banners
   - **Q3 - Mes Morado**: Lord of Miracles themed, purple visual filter, exclusive religious-themed events
   - **Q4 - Navidad Criolla**: Christmas themed, gift-exchange deal variant, exclusive holiday portadas
   - Each season has a battle pass style progression track with 30 tiers of cosmetic rewards

4. **"Propiedad del Dia"** (daily FOMO):
   - One premium property added to the market deck each day that isn't normally available
   - Higher base rent, unique visual design
   - Only appears in matches started on that calendar day
   - Creates daily reason to log in and start a match

5. **Streak Protection with FOMO**:
   - The existing streak system (spec 3.5.5) resets after 48h without login
   - Add: Streak Freeze item earned during limited events — lets you skip one day without losing your streak
   - This turns the Streak Freeze into a scarce, valuable commodity that players hoard

---

## Cross-Cutting Principles

### The Viral Flywheel

Every mechanic above feeds into a single loop:

```
PLAY → EXPERIENCE EMOTIONAL PEAK → SHARE (portada/clip/invite) →
FRIEND SEES → DOWNLOADS → PLAYS → EXPERIENCES PEAK → SHARES → ...
```

The job of every design decision is to reduce friction at each step of this loop.

### Key Metrics to Track

| Metric | Target | Why |
|--------|--------|-----|
| Share rate (portadas shared / matches played) | >5% | Organic acquisition engine |
| K-factor (installs from shares / total shares) | >0.3 | Viral coefficient |
| Time to first share | <3 matches | Player must experience a shareable moment quickly |
| Referral completion rate | >15% | Referred friends who complete tutorial |
| Watcher-to-Owner conversion | >20% | Spectators who start playing |
| Daily active / monthly active ratio | >25% | Retention driven by FOMO + social hooks |
| Average session count before first invite sent | <5 | Social loop activates early |

### What NOT to Do

1. **Do not gate core gameplay behind social mechanics.** Friends should make the game better, not playable. Solo players must have a complete experience.
2. **Do not spam invite prompts.** One prompt at an emotional peak per session maximum. Respect the player's social capital.
3. **Do not make FOMO punitive.** Missing an event should feel like missing an opportunity, not losing progress.
4. **Do not make share cards cluttered.** One vanity number, one visual narrative, one CTA. Anything more reduces share rate.
5. **Do not force sharing.** Every share prompt must have a visible, frictionless dismiss option. Forced sharing creates resentment.

---

## Sources

- [Monopoly GO Social Systems Analysis (Mobile Game Doctor)](https://mobilegamedoctor.com/2025/05/30/social-systems-in-monopoly-go/)
- [Monopoly GO User Acquisition & Monetization (Fungies.io)](https://fungies.io/monopoly-go-a-masterclass-in-user-acquisition-monetization-and-player-psychology/)
- [Social + Competition: How Monopoly GO Keeps Users Engaged (ChatsLine)](https://www.chatsline.com/blogs/view/6046/social-competition-how-monopoly-go-keeps-users-engaged-long-term)
- [The UX Behind Fall Guys (Medium)](https://medium.com/ux-for-games/the-ux-behind-fall-guys-how-a-game-can-be-simple-yet-engaging-5602f67b1151)
- [Everything You Can Learn From Fall Guys (GameAnalytics)](https://www.gameanalytics.com/blog/everything-you-can-learn-from-fall-guys-ultimate-knockdown)
- [Among Us - Wikipedia](https://en.wikipedia.org/wiki/Among_Us)
- [Addressing Conflict: Tension and Release in Games (Gamasutra)](https://www.gamedeveloper.com/design/addressing-conflict-tension-and-release-in-games)
- [Engines of Experience - Designing Games (O'Reilly)](https://www.oreilly.com/library/view/designing-games/9781449338015/ch01.html)
- [Emotional Peaks in Game Design (Medium)](https://medium.com/design-bootcamp/the-catharsis-of-choice-how-emotionally-intense-experiences-in-games-transform-player-engagement-e791c449ac31)
- [How to Grow Your Game with Referrals (Gamasutra)](https://www.gamedeveloper.com/business/how-to-grow-your-game-with-referrals)
- [Power-up UA with a Friend Referral Program (GameRefinery)](https://www.gamerefinery.com/power-up-ua-with-a-friend-referral-program/)
- [Referral Marketing for Mobile Games (Megacool)](https://megacool.medal.tv/blog/referral-marketing-for-mobile-games-how-to-grow-mobile-game-with-referrals/)
- [10-Step Guide to Referral Marketing in Gaming (WebEngage)](https://webengage.com/blog/referral-marketing-for-gaming/)
- [FOMO as Behavioral Manipulation in Games (Medium)](https://medium.com/@milijanakomad/product-design-and-psychology-the-exploitation-of-fear-of-missing-out-fomo-in-video-game-design-5b15a8df6cda)
- [How to Retain Players Through FOMO (EntheosWeb)](https://www.entheosweb.com/how-to-retain-players-through-fomo-in-game-design/)
- [Timegated Content in Mobile Gaming 2025 (Macho Levante)](https://macholevante.com/news-en/182194/unlocking-the-power-of-timegated-content-the-secret-weapon-in-mobile-gaming-engagement-2025/)
- [Impact of Seasonal Events on Game Marketing (The Game Marketer)](https://www.thegamemarketer.com/insight-posts/the-impact-of-seasonal-events-on-game-marketing)
- [Spectator Mode Design (Lark)](https://www.larksuite.com/en_us/topics/gaming-glossary/spectator-mode)
- [Mapping Design Spaces for Audience Participation in Streaming (ACM)](https://dl.acm.org/doi/fullHtml/10.1145/3411764.3445511)
- [Expanding Player Impact in Social Spaces (Polaris)](https://polarisgamedesign.com/2024/expanding-player-impact-in-social-spaces/)
- [Beyond Gameplay: Casual Games Competing in 2025 (FoxData)](https://foxdata.com/en/blogs/beyond-gameplay-what-casual-games-are-competing-on-in-2025/)
- [Viral Gameplay in 2026: Live Gaming Clips (TechTimes)](https://www.techtimes.com/articles/313453/20251218/viral-gameplay-2026-why-live-gaming-clips-dominate-youtube-shorts-tiktok-feeds.htm)
- [TikTok Shaping Mobile Game Trends (GeniusCrate)](https://www.geniuscrate.com/the-explosion-of-short-form-gaming-how-tiktok-is-shaping-mobile-game-trends)
- [Chess.com Game Review Features](https://www.chess.com/news/view/chesscom-launches-game-review-v2)
- [Brawl Stars Battle Cards (Fandom Wiki)](https://brawlstars.fandom.com/wiki/Battle_Cards)
- [Brawl Stars Star Player (Supercell Support)](https://support.supercell.com/brawl-stars/en/articles/star-player-2.html)
- [Rubber Banding vs Snowballing (BoardGameGeek)](https://boardgamegeek.com/thread/904667/rubber-banding-vs-snowballing-catch-up-vs-stay-ahe)
- [Catch-Up Mechanisms (The Thoughtful Gamer)](https://thethoughtfulgamer.com/2017/03/28/catch-up-mechanisms/)
- [Social Features in Mobile Games (Udonis)](https://www.blog.udonis.co/mobile-marketing/mobile-games/social-features-mobile-games)
- [The Future of Sharing for Mobile Games (Megacool)](https://megacool.medal.tv/blog/the-future-of-sharing-for-mobile-games/)
- [Top Social Features in Mobile Games (Helpshift)](https://www.helpshift.com/blog/top-mobile-games-that-have-social-features-incorporated/)
