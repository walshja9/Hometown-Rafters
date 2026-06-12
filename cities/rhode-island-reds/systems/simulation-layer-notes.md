# HOMETOWN RAFTERS — RHODE ISLAND REDS
## Simulation Layer Design Notes
### COLLECTED FEEDBACK & BLUEPRINTS — TO BE BUILT

---

## STATUS: PARKED FOR FUTURE DEVELOPMENT SESSION

This document collects all mechanical and simulation-related feedback from design reviews. These are not finalized systems — they're blueprints and principles to build from when we shift from narrative to engineering.

---

## 1. THE THREE-PATH MANAGEMENT DIVERGENCE

The Roots, Ambition, and Profit paths must play as three mechanically distinct management games, not three narrative skins on one sim.

### Roots Path: The Closed-Loop Engine
- Zero free agent budget. Competitive advantage is purely internal player development.
- Players drafted from local New England area get a massive passive buff to development rate (the "Gus Pipeline" mechanic).
- Plays like a drafting and service-time management challenge.
- Nobody wants to trade with a broke mill-town team. Trade market is severely limited.
- Satisfaction comes from squeezing a 90-win season out of a roster that costs nothing.
- Gus is your lifeline — the only competitive advantage you have.

### Ambition Path: The Standard Simulator
- Traditional budget. Normal trade mechanics. Standard evaluation metrics.
- Player manages ticket prices, stadium upkeep, balances veterans with rookies.
- Plays like a traditional dynasty mode relying on actual baseball acumen.
- Baseline "vanilla" experience against which the other two paths feel different.

### Profit Path: The Compromised Database
- Flush with cash early. Winning is easier in the short term.
- Front office logic gets hijacked by connected ownership — forced signings, bloated contracts, roster spots for connected players.
- Player deals with sudden inputs that bypass standard roster management.
- Slowly corrupts farm system and cap space.
- Plays as crisis control — keeping an unstable roster from collapsing under its own weight.
- Gus becomes an obstacle to forced high-priced moves.

**Key Principle:** You don't need to change a single word of Gus's dialogue for the player to feel entirely different about him on each path. The mechanical context changes the emotional relationship.

---

## 2. ON-FIELD MOMENT SYSTEM MODIFIERS

### The Runtime Formula
```
Effective_Stat = (Base_Stat × Era_Multiplier) + (Culture_Modifier × Stakes_Multiplier)
```

Culture modifiers should affect Moment gameplay without permanently changing player ratings. A 70 OVR contact hitter is still 70 OVR. But in a Moment, the environment bends the odds.

### Corrupt Path: "Intimidation" Buff
- Pitcher velocity buff and inside-pitch efficacy at home.
- Umpire strike zone slightly expands for your pitchers at home (Federal Hill Rules).
- Trade-off: Discipline debuff. Hitters chase more. Pitchers throw more wild pitches.
- Team plays aggressive but reckless. Winning ugly.
- Era multiplier strongest during Golden Age.

### Clean Path: "Clutch" Buff
- Stamina and contact buff when trailing in 7th inning or later.
- RNG favors finding gaps. Team refuses to strike out looking.
- Plumber's approach — put the ball in play, trust the work.
- Era multiplier strongest during Reckoning.

### Era-Specific Feel
- **Dead Ball (1918–1930):** No home runs. Contact and speed only. Bunting is primary weapon. Complete games are norm. Chess-like — manufactured runs, stolen bases, sacrifices.
- **Transition/Golden Age (1946–1975):** Game opens up. Power viable. Faster, more aggressive. Corrupt intimidation buff most powerful here.
- **The Fall (1976–1992):** Moments feel desperate and lonely. Empty stadium, thin crowd noise. Emotional payoff for winning is enormous because you did it with nothing.
- **Reinvention/Reckoning (1993–Present):** Analytics era. Shifts, bullpen matchups, launch angle. More information, more tools, more pressure. Clean clutch buff most powerful here.

---

## 3. THE TRADE & EVALUATION ENGINE

### Shadow Economy Effects
- **Investigation Clock active:** Opposing GMs devalue your prospects. Assume your pipeline is tainted. A prospect worth 80 in a clean trade returns 60 on the corrupt path.
- **Favor Evaluation:** When mob demands a specific player acquisition, the selling team recognizes maximum leverage. Automatic overpay, bleeding the farm system.
- **Clean Path Bonus:** Word of mouth reputation. Well-run franchises get slightly better trade offers because other GMs trust the development and scouting reports.

---

## 4. THE CORRUPTION TAX MECHANICS

### The Favor Economy (Escalating Demands)
**Early (Era 2–3):**
- Nephew roster spot
- Connected vendor gets concession contract
- Luxury box built from player budget
- Forced prospect evaluation for somebody's kid

**Escalating (Era 3–4):**
- Star player forced to make appearances at Federal Hill
- Connected manager candidate pushed by ownership
- Farm system budget redirected to "business opportunity"
- Game management interference (bullpen usage for specific outcomes)

**Crisis (Era 4–5):**
- Player implicated with bookmakers (cover up or go public)
- FBI investigation — books examined, deep corruption = catastrophic exposure
- Connected ownership wants to sell stake to another connected buyer
- Witness intimidation requests

### The Volatility Tax (Systemic Instability)
- Budget fluctuates randomly. Some years extra cash, other years chunks disappear. "Business expenses."
- Connected players have uncontrollable off-field lives. Missed weeks, showing up out of shape.
- Front office leaks. Trade discussions, scouting reports leaked to other teams.
- Paranoia factor — player can't tell which NPCs report to ownership vs. loyal to franchise.

### The Investigation Clock
- Invisible probability that increases with volume of corrupt transactions.
- Warning Phase: Media questions, beat writer articles, ownership tells player to handle it.
- Active Investigation: Feds involved. Transactions scrutinized. Some moves locked.
- Indictment: Owner goes down. Severity depends on depth of corruption.

### Corruption as Loss of Agency (Progressive)
- Early corruption costs money.
- Mid corruption costs roster spots.
- Late corruption costs decisions — connected manager auto-sims games, auto-sets lineups, locks player out of Moment System on certain days.
- The mechanical experience of powerlessness is what makes the player regret taking the money.

### Anti-Optimization Rules
- The Nephew MUST be on 25-man active roster and cannot be benched more than 3 consecutive games.
- Budget tax hits AFTER allocation — can't plan for it, forces operating in the red.
- Connected manager occasionally locks player out of decisions entirely.

---

## 5. THE DIEGETIC GUS STANDARD — UI INTEGRATION

### The Dashboard Problem
Whisper signals can't only exist in a 3D stadium view. They must bleed into the management UI where the player actually spends time.

### UI Reflections of the Gus Standard
- **High Standard:** Player profiles include hometown details, personal notes from scouts who know the kid's family. Loading screens are hand-drawn Bandbox sketches. Interface feels warm, personal.
- **Low Standard:** Player profiles are pure stats. No stories. No context. Just numbers. Loading screens are slick, sterile corporate graphics. Interface feels cold, efficient.
- **GM Desk:** Cluttered with unread legal notices when things are bad. Clean and organized when things are good. Phone rings with no one on the other end during Investigation Clock phases.

---

## 6. DANNY COUTURE — TRADE DEADLINE PROTECTION

Danny cannot be hard-locked from trades. But trading him during the Fall triggers:
- The old-timer leaves the parking lot. Permanently.
- Attendance drops from ~3,000 to near zero.
- Seventh Inning Salute stops happening.
- Gus Standard takes massive hit.
- Reinvention era starts from a deeper hole — community that would have supported rebuild watched franchise sell the one guy who chose to stay.

The spreadsheet says trade him. The world says don't. That tension IS the Fall era.

---

## 7. REPLAYABILITY ARCHITECTURE

### Core Principle
The emotional depth of seeing Gus react differently is NOT enough to carry a 35-year replay alone. The management sim must feel mechanically distinct between paths.

### What Makes Replay Work
- Three different management games (Roots/Ambition/Profit) with fundamentally different economic loops.
- Same Gus, same dialogue, completely different relationship to the player's mechanical experience.
- Path-dependent legends as replay reward — you replay to see who shows up.
- Emergent triggers (Boston Traitor, Investigation Clock) tied to simulation state, not calendar — two corrupt playthroughs feel different.
- Legends gated by Gus Standard in later eras — low standard means no Reckoning legend, which drives desire to replay and do it right.

---

## NEXT STEPS WHEN WE BUILD THE SIM LAYER
1. Define exact attribute modifiers for each era (Dead Ball through Modern)
2. Build the Favor Economy event table with probability weights
3. Design the Investigation Clock probability curve
4. Map the diegetic UI changes to specific Gus Standard thresholds
5. Define the Gus Pipeline development rate buffs
6. Build the trade evaluation shadow economy math
7. Define era-specific scouting mechanics (guy who knows a guy → data pipeline)
8. Stress-test all systems against min-max player behavior
