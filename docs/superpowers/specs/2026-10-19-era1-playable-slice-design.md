# Hometown Rafters: Era 1 Playable Slice — Design

**Date:** 2026-10-19
**Status:** Approved in brainstorming, section by section. Awaiting Alex's review of this written spec.
**Inputs:** Two independent positions written before either saw the other: `inputs/2026-10-19-fable-position.md` and `inputs/2026-10-19-astra-position.md` (Codex gpt-6-astra). This spec is the synthesis Alex approved. Where it differs from either input, this spec wins.

## 1. Goal

Turn the Rhode Island Reds design docs into a sellable game. The first build is a playable slice of Era 1, the Mill Era, 1918–1930. It has two jobs:

1. Prove the loop: the front office, simulated seasons, managed story games, Moments and the ballpark walk all work together.
2. Prove the hook: a player who never sees a number can still feel the Gus Standard change, and can explain *why* the park changed.

## 2. Locked decisions

| Decision | Choice |
|---|---|
| Game form | Narrative management fused with real team management. Story choices arrive as cards. Roster moves feed the hidden meters. |
| Play depth | Hybrid. You run the front office every season and games are simulated. Two story games a season are handed to you to manage inning by inning. Moments are rare at-bats played pitch by pitch. |
| Look | OOTP-style tables for roster and team. Period-styled text (newspaper, scorecard, ledger) and a simple diamond view for managed games and Moments. |
| Later, not now | Arcade at-bats (2D, Retro Bowl style, never 3D), the Steam wrapper, Era 2. |
| Commercial intent | Possibly sold on Steam. TypeScript web app first, wrapped with Electron for Steam later. |
| Scope | Era 1 only, 1918–1930, 13 seasons. 1930 belongs to Era 1; Era 2 starts in 1931. |
| Approach | Thin whole loop. Build every layer thin at once, then thicken from playtests. Prove one season, then three, before writing all thirteen. |
| Who builds | Fable implements directly. No Codex (Alex, 10/19). |

The player is the franchise itself, with no avatar (`structural-fixes-audit.md` FIX 4). Gus reacts to teammates and the club, never to a GM character. A Moment drops that framing for one at-bat.

## 3. The history rule

Real history supplies the world. The fictional league replaces real baseball's teams and players.

- **Kept real:** the mills and the textile collapse, the immigrant neighborhoods, the laws, prices and wages, and baseball's actual rules and run environment, including the 1920 spitball ban and the cleaner, livelier ball.
- **Fictional:** the eight-team league, its clubs, owners and players. No real ballplayers appear.
- **Facts checked on 10/19:**
  - Coffee milk dates to the early 1930s; the first bottled syrup was 1926. It is not in Era 1.
  - The Jacobs Brothers (later Sportservice, founded 1915) lent money to struggling clubs in exchange for concession rights. The Era 1 Money path is grounded in that.
  - The 1920 average major-league salary was about $5,000.
  - A 1925 bleacher seat at Yankee Stadium cost 55¢.
  - Branch Rickey's Cardinals pioneered the farm system in the early 1920s, so the St. Louis Perfectos can invent it inside the slice.
- **Unverified:** Rhode Island's Sunday-baseball legalization date. The review's "1919" is unsourced; only the Rocky Point toleration is confirmed (SABR). Sunday ball stays Era 1's fight, but no date goes into content without a primary source.
- **Must be handled on purpose:**
  - The color line. Baseball was segregated in 1918–1930, and the Negro National League was founded in 1920. Rhode Island's Cape Verdean community sat in complicated relation to that line.
  - A native, ideally Azorean, check of Gus's Portuguese before release.
  - The review's Era 1 history fixes, applied while converting docs into content.

## 4. The season loop

One season takes 30–40 minutes, so the slice runs about 7–9 hours.

1. **Winter.**
   - The Coalition's budget letter arrives.
   - Renew contracts.
   - Buy, sell or trade players.
   - Set ticket prices and fund repairs.
2. **Opening Day.** Set the lineup, defense, rotation and bench, then face the first choice card.
3. **The season.** It plays in three blocks. News, standings, transactions and two more choice cards come between the blocks.
4. **Story games.** Two per season are managed inning by inning; either can be simmed instead. A pennant-deciding game takes one of the two slots.
5. **Moments.** Most seasons have none. About six are authored across the era. An elimination Moment fires only when a real situation arises: a game the Reds must win to stay alive, inning 7 or later, with the tying or winning run at bat or on base. Scores are never manufactured to create one.
6. **Fall.** The books are settled. Then comes the ballpark walk: three short scenes, ending on next year's immediate problem.

**Era 1's win condition is survival** (`era-map-championships.md:131`). The Reds start as the league's poorest club with a thin roster. The sim never rigs games. The docs' "zero championships" therefore becomes "very unlikely": a sharp front office can still steal a pennant.

## 5. State model

**Visible:** cash, debts, attendance, standings and the roster.

**Hidden (all 0–100).** The player never sees these numbers or the band names.

| Value | Start | Moves |
|---|---|---|
| Gus trust | 65 | Keep a promise +2. Break one −5. Deliberate exploitation −12. Losses never move it. |
| Culture | 50 | Inclusive practice +1 per qualifying event, capped at +4 a year. Neglect −3. Serious betrayal −8. |
| Gus Standard | derived | `0.4 × trust + 0.6 × culture`. Four bands: 0–24, 25–49, 50–74, 75–100. Culture outlives Gus, so the Standard keeps its meaning after his exit. |
| Fan loyalty | 50 | Affordable admission +2 a year. Exclusionary admission −2 a year. A broken community promise −6. The season's result moves it by ±1 at most. It drives attendance only. |
| Infrastructure | 65 | Shown physically, never as a number. Skipped maintenance −8 a year. One repair unit +10. Below 40, sections close and capacity falls. |
| Path drift | 0 each | Roots, Ambition and Money scores. Each winter all three are multiplied by 0.85. A normal decision adds 6 toward its path and a landmark decision adds 12. Recurring policies count once a year. |

**Path momentum:** `m = (leading score − runner-up) / 100`. Points toward any path other than the leader are divided by `1 + m`. The leading score is the path, with ties broken by the most recent substantive choice. Scores are clamped to 0–100. A path never locks a choice away. The 1930 Foundation is accumulated identity, not a final menu.

**Readout rules:**
- **Hysteresis:** a band readout flips only after the Standard crosses a band edge by 3 points.
- **Timing:** Gus reacts immediately, through how early he arrives, how much he talks, and which Portuguese he uses (warm phrases, or "cabeça de mula", `gus-almeida.md:45`). The crowd changes at season boundaries.
- **Era-true signals only:** the same families in their seats or missing, vendors extending credit or not, and the Seventh Inning Salute being born and taking hold. There is no coffee milk and no parking-lot old-timer in Era 1.
- **Causal readouts beat band readouts.** For example: ticket prices raised within two seasons, and loyalty below 45, means the family from 1918 is missing.

**Honesty rule:** culture changes opportunities, relationships and atmosphere. It never changes batting or pitching outcomes.

**The Money path in Era 1 is not organized crime.** It means sponsor and concession deals that pay well but restrict admission or concessions. Obligations persist when the path changes: turning toward Roots never cancels a signed contract.

All numbers in this section are starting values, to be tuned by the strategy bots (§11).

## 6. League and players

- **Teams and schedule:**
  - Eight founding cities (README:154). Each plays a 154-game schedule, 22 games against each rival.
  - It is a single league: the pennant winner is the champion, and there is no postseason series in the slice.
  - The six cities besides Rhode Island and St. Louis get light identities only: city, club name, owner personality, budget tier and park size. The docs already imply Boston.
- **Rosters:**
  - 25 active players per club, with no farm system.
  - Players move by trade, by cash sale, or by purchase from a pool of abstract independent minor-league clubs with asking prices.
  - Local amateurs sign first contracts, which is the Roots pipeline.
- **Ratings (0–99, 65 = league average):**
  - Hitters: contact, power, eye, strikeout avoidance, speed, fielding, arm.
  - Pitchers: control, strikeout stuff, contact suppression, stamina.
  - Everyone: hidden potential, age, handedness (a simple platoon multiplier).
- **Development:**
  - Newcomers are 17–23.
  - Skills grow about +1 to +3 a year through about 27, hold steady, then decline about −1 to −3 a year after about 30. Speed declines earlier.
  - The randomness is seeded.
  - Potential caps growth. Playing time matters, but benching never destroys talent.
- **Contracts:** one year at a time under the reserve clause. Players hold out if an offer falls well below their expectation. Cutting a local player's pay costs loyalty or culture.
- **Gus is authored and cannot be sold in this slice.** In-world, the Coalition promised the neighborhood he stays, and the game says so. His statistics come out of his ratings and are never forced.

## 7. Rival owners and the ML policy

Each of the seven rival owners has:
- a personality (for example a cheapskate, a gambler or a loyalist);
- a budget tier;
- a ledger of every deal with the Reds, including how fair each was against the valuation model and any broken deals.

Patterns in the ledger change their behavior:
- If you sell a star every winter, they wait you out and lowball.
- If you fleece one, he stops taking your calls.
- If you're a long-term fair partner, he offers you first look at his best young player.

These are deterministic rules, so they can be tested and balanced. This is the slice's answer to Alex's "AIs that learn off your decisions".

**Machine learning is used on the development side only.** Nothing in a player's save learns at runtime: thirteen seasons is too few decisions to learn from, and the results would be untestable.

- **Valuation:** M2 ships a hand-written valuation formula. A model trained on simulated seasons replaces it only if the bots show AI GMs making bad trades.
- **Balance bots:** strategy bots may later learn to hunt for exploits.
- Any learned results ship frozen.

## 8. Money

The real figures come from a research pass using SABR salary and finance data. Then the bots tune them. The scale anchors are a roughly $5,000 average salary in 1920, bleachers near 50¢, and a Bandbox of 10,000–12,000 seats. The Reds start as the poorest club, and break-even is all the Coalition expects (`ownership-lineage.md:28,35`).

- **Income:**
  - The gate: attendance × price, capped by usable capacity.
  - Concessions, or a concession contract in their place (the Jacobs Brothers–style Money deal).
  - Scorecard and fence advertising.
  - Player sales.
  - Coalition support.
- **Attendance:** demand comes from price, loyalty (±1% per point away from 50), results (±15% at most), Sunday dates and capacity.
- **Sunday baseball is the economic hinge.** Sunday was the mill workers' one day off, so winning that fight unlocks the biggest crowds of the week.
- **Expenses:** payroll, rail travel, park upkeep, league dues, and the visiting club's share of the gate.
- **Safeguards:**
  - Every trade and sale runs through one valuation model with a transaction spread, so buying low and selling high can't manufacture money.
  - Buyers must need the player and have the cash.
  - Loans are capped, charge interest, and amortize over three seasons.
- **The era's end:** Ashworth's mill closes and Coalition support ends in 1929; demand falls in 1929 and again in 1930.
- **Failure:** if obligations can't be met after legal borrowing and sales, the run ends with a foreclosure headline. Every path must have a viable route through the downturn.

## 9. Baseball engine

- **Adapt, don't port, `sim_engine.py`** (`D:\Claude Context\MLB The Show 26 - Reverse Engineer Player Ratings\`).
  - **Keep:** the ratings-to-tendency math, log5 matchups and the injected RNG.
  - **Remove:** the modern constants.
  - **Fix two verified bugs.** Strikeout, walk and hit-by-pitch odds are computed independently, so their sum can exceed 1. And `simulate_half_inning` has no walk-off stop.
- **One pitch-level engine for every mode.** Log5 sets each plate appearance's target outcome odds for the season's environment. A count-transition model plays the pitches so that, on average, outcomes hit those targets. One league season is about 175,000 pitches, which a browser handles in seconds. Simmed games, managed games, Moments and the later arcade mode all call the same code, so identical decisions give identical results in any mode.
  - **Simmed games:** an AI manager plays period-style baseball. Starters go deep, complete games are common, and bullpens are small.
  - **Managed games:** the engine pauses between plate appearances for steal, bunt, hit-and-run, pinch hitter, pitching change and intentional walk.
  - **Moments:** the engine pauses on every pitch for take, swing, protect, bunt or sit on a pitch type. Correct reads improve the odds and wrong ones hurt them. Average reads land near the simmed odds, so Moments cannot become a free win.
  - **Later arcade mode:** it adds timing input through an adapter. The same rules still decide advancement, outs and runs.
- **Era environments.** There is one record per season, 1918–1930. It holds K, BB, HBP, HR, 2B, 3B, errors, SB and runs per game, fitted to real league averages (Retrosheet, Baseball-Reference). The years run from dead ball in 1918–19, through the 1920 jump, to home runs climbing through the twenties. Strikeouts stay under 10% throughout, and errors are far above modern rates.
- **Rules:**
  - The engine tracks which runner is on which base, so speed and arm matter.
  - Base advancement depends on the type of hit.
  - It handles steals and caught stealing, sacrifices, double plays and errors.
  - Walk-offs end the game on the spot, and extra innings are supported.
  - Pitchers tire, and they bat (no designated hitter).
  - Substitutions are legal, and the engine tracks the batting order and pitcher workload.
- **Outputs:** box scores, standings, league leaders and newspaper events, all built from one event stream.
- **The ratings converter stays a dev tool** (for test players and sanity checks). It never ships.

## 10. Content and scheduling

**Choice cards:** one Markdown file per card, with YAML frontmatter and the prose as the body. The frontmatter holds:
- the year window and phase (winter, opening, between blocks, fall);
- prerequisites, priority and repeat policy;
- the options and their follow-ups.

Effects come from a fixed vocabulary with no code:
- `spend` / `earn`;
- `adjust` (a hidden value, by an amount);
- `set_flag`;
- `obligation` (a contract or promise with a due date);
- `transfer` (a player);
- `followup` (a card, N seasons later).

An illustrative card (the shape, not final content):

```markdown
---
id: sunday-ball-01
years: [1918, 1919]
phase: winter
requires: { not_flag: sunday_ball_settled }
priority: 80
repeat: never
options:
  - id: petition
    text: Back the junior partners' petition for Sunday games.
    effects:
      - { adjust: culture, by: 1 }
      - { adjust: drift.roots, by: 6 }
      - { spend: 300 }
      - { followup: sunday-ball-02, in_seasons: 1 }
  - id: defer
    text: Tell them Ashworth won't hear of it this year.
    effects:
      - { adjust: loyalty, by: -2 }
      - { followup: sunday-ball-01b, in_seasons: 1 }
---
The bakery owner keeps the petition folded in his apron pocket...
```

**Readouts** form a table. Each entry has a place (gate, bleachers, clubhouse, concession, Gus), a year span, conditions (band, or a cause such as a recent price rise plus low loyalty) and prose variants.

**Moments** are one file each. A Moment file holds:
- eligible games, participants and the trigger;
- setup prose and per-pitch narration hooks;
- the aftermath for each outcome.

**Scheduler.** At each phase, slots are filled in this order:
1. Fixed historical events, such as the 1920 rule changes and the 1929 mill closing.
2. Consequences that have come due.
3. Eligible cards, chosen by weight.

Mandatory consequences take optional slots. Every card has at least one affordable option, even if it hurts.

**Checker.** Every build validates the content against the schema. Errors read in plain English, such as `card sunday-ball-02 uses unknown flag "sundy_ball"`.

**Doc mapping:**
- Era 1 of `choice-tree.md` becomes the cards: ticket assistance, the kid needed at the mill, repairs, sponsorship, and trading a founding favorite (`choice-tree.md:49`).
- `ownership-lineage.md` becomes the Coalition's winter letters.
- `gus-almeida.md` supplies Gus's reactions and Portuguese.
- `diegetic-standard-corruption.md` sets the readout rules.
- `era-map-championships.md` sets the stakes and Moments.

**Volume:**
- About 40–60 cards, about 150 readout entries, six Moments, and headline templates.
- Around 40,000 words in total.
- Fable drafts from the docs in their established voice (per the repo's CLAUDE.md), and Alex edits.

## 11. Architecture, saves, tests

- **Repo:** `D:\CodexWorkspaces\hometown-rafters-game`, with a private GitHub remote. The docs repo stays the canon bible. The game repo holds the code plus the content converted from the docs.
- **Stack:** TypeScript, Vite, React and Vitest. The browser build comes first; Electron comes later.
- **Core, pure TypeScript with no DOM:**
  1. `baseball`: the pitch engine, era environments and the AI manager.
  2. `league`: teams, players, development, the schedule, transactions, valuation and rival owners.
  3. `franchise`: money, hidden values, drift and obligations.
  4. `story`: content loading and checking, the scheduler, effects and readout selection.
  5. `save`: serialize, deserialize and migrate.
- **UI:** React screens (tables, newspaper, diamond, ballpark walk) on top of the core. A Web Worker is added only if simulation stalls the UI.
- **Randomness:** seeded, with separate streams for baseball, development, scheduling and flavor text. Viewing a screen never changes results.
- **Saves:**
  - Contents: the full state, each RNG stream's position, the rules version and the content version.
  - When: autosave at every phase boundary, and on every pitch in managed games and Moments.
  - On failure: if a load fails, the last good save is kept.
  - Where: browser storage now, with file export. Files on desktop later.
- **Tests:**
  - **Rules:** probabilities sum to 1; runs are conserved; walk-offs, extra innings and substitutions behave.
  - **Determinism and mode-equivalence:** the same seed gives the same season; simmed and managed play give identical results for identical decisions.
  - **Calibration:** batches of simulated seasons land within tolerance of each year's real averages.
  - **Content:** every card passes the schema check and is reachable on some path.
  - **Strategy bots:** all-Roots, all-Ambition, all-Money, a flip-flopper, a price-gouger, a sell-everything owner and a last-year borrower each play 1918–1930 headless. Report pennants, cash and Standard. If one strategy dominates, fix the numbers before writing more content.
  - **Human playtests:** the key question is whether players can explain why the park changed.

## 12. How 1930 ends

- **The last walk** revisits three things established in 1918: a family, a concession spot and Gus's routine. Their condition is the emotional verdict.
- **A newspaper retrospective** covers the thirteen seasons: pennants, memorable players, cash and debts.
- **Three verdicts, earned separately:**
  - Baseball: what was won.
  - Finances: obligations paid, a legal 25-man roster, infrastructure at least 40, and positive cash net of debt, with the threshold set in the money research.
  - The town: told in prose, never as a score.
- **The path** is described as who the franchise became.
- **Gus is 30 and still playing shortstop.** His decline is not spent here.
- **Closing screen:** what carries forward into 1931 and the Depression.
- **Early end:** foreclosure ends a run before 1930.

## 13. Build order

1. **M1 — Baseball core.** One game, simmed or played pitch by pitch on a bare diamond. It scores correctly, saves and reloads mid at-bat, and has its rule tests.
2. **M2 — League.** Eight generated teams play a full 154-game season, with standings, box scores, basic transactions, the valuation formula and the calibration tests.
3. **M3 — One complete 1918.** The budget letter, roster moves, cards, two story games, a Moment, settlement and the ballpark walk, then 1919. The screens are plain.
4. **M4 — Three seasons (1918–1920),** including the 1920 shift. The strategy bots run the full 1918–1930 with placeholder content, plus a 1929–30 money stress test. **Go/no-go gate:** is it fun, and can playtesters read the Standard?
5. **M5 — The rest of the era.** Content for 1921–1930, the period look and polish.

Each milestone gets its own implementation plan, written when the previous milestone lands. The first plan covers M1 only.

After the slice: the Electron wrapper for Steam, arcade at-bats, then Era 2.

## 14. Risks and cut list

1. **The story and the baseball feel like two games.** Every card must touch a roster, an account or a person; cut unrelated cards first.
2. **One strategy dominates.** The bots catch this before content is written.
3. **The hidden state feels arbitrary or too readable.** Use causal readouts and playtests, and cut decorative variants before clarity.
4. **Writing volume.** Prove one season, then three, before writing thirteen.
5. **OOTP feature creep.** No scouting uncertainty, complex injuries or negotiation minigames in the slice. Keep fatigue, defense, running and substitutions.
6. **History as costume.** The research pass, the color line and the Portuguese check all land before release.
7. **The team name.** "Rhode Island Reds" was also used by a long-running hockey team. Check the trademark before selling.

**If it runs long, cut** in this order: two story games down to one; simpler rival owners; the minor-league purchase pool; the Electron wrapper. **Never cut:** the thirteen seasons, real transactions, the ballpark walk, or the shared Moment engine.

## 15. Open items (resolve during the build, not blockers for M1)

- Names, owners and budget tiers for the six other founding cities (before M2).
- The economic research pass: real 1918–1930 salaries, ticket prices, attendance and expenses (before M3).
- A primary source for Rhode Island Sunday baseball, and whether the slice uses the real date (before writing the Sunday cards).
- How the fictional league handles the color line in Era 1 (before M5 content).
- The Era 1 card list drawn from `choice-tree.md`, with the review's history fixes (M3–M5).
- The Portuguese check and the trademark check (before release).
