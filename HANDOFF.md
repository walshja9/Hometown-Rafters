# Handoff: Hometown Rafters (2026-10-21, click-through fix wave)

## Latest (10/21): Alex's M3 click-through done; fix wave pushed (game repo main at bb92ddd, 406 tests)
- Alex's notes: it felt good; (1) no way to compare a player's stats or ratings across years when trading or releasing; (2) scenarios ran out after three or four seasons and "sim to next event" found none; (3) first place winning outright with no playoff feels weird.
- Fixed in 5774606 + bb92ddd: `L.seasons` archives every year's stats (with end-of-season overall) and the Deals page shows a career table for the two picked players (save v5, no migration); `value()` uses age instead of hidden potential under 27 (the price leak); every card runs to 1930 (the fixed 1919 Salute excepted) plus two yearly low-weight cards (`the-grounds-man` winter, `opening-day-band` opening); `bleachers` and `clubhouse` readouts written and the walk now renders five scenes.
- Still open from the notes: the 1922-1930 deck proper (M4 content); the pennant is silent (no clinch event, letter line or readout), so a pennant win feels automatic. Fable's recommendation: keep pennant-only for era 1 (minor leagues crowned the pennant winner outright; Shaughnessy playoffs arrive 1933 = era 2, a story the player can live through) and add a clinch event in M4; a split-season playoff is the era-true alternative if Alex wants one, and it is a plan item, not a fix. Awaiting Alex's call.

## M3 (one season as a franchise) BUILT, reviewed and pushed
- Game repo `D:\CodexWorkspaces\hometown-rafters-game` main at 36e1d56, 396 tests green, build clean. Built by Codex (one dispatch per task, write-capable, cwd = the repo) with Fable reviewing and committing each task; Alex lifted the "no Codex" rule on 10/20 ("loop in a codex agent"). Codex's sandbox cannot write `.git` or spawn Vite's default loader, so it verifies with `npm test -- --configLoader native --pool threads` and `npm run build -- --configLoader native`, and the reviewer commits. Ledger and reports: `docs/m3-build/` in the game repo.
- What M3 is: the season as a franchise. Winter office (the Coalition's letter, contracts, ticket price, repairs, loans, the winter card), Opening Day card, three season blocks with a card at each boundary, two story games (Opening Day; the pennant decider or the last home game), an elimination Moment (Gus, inning 7+, tying run on, a loss ends the race) played pitch by pitch with a count-aware read model, the fall books with loans and foreclosure, the ballpark walk (gate, concession spot, Gus), then the next winter. Hidden values move only through choices and policies and are read back only as prose. Content: 13 cards, 30 readouts, one Moment, letters 1918-1930, three story-game texts under `content/`, checked at test time.
- Plan: `docs/superpowers/plans/2026-10-20-m3-one-season.md` with "Execution notes" (every plan correction found in the build, including the Moment calibration rewrite and the probe fix wave).
- Final review: Codex adversarial review of the whole branch (`docs/m3-build/final-review.md`): 7 findings, all fixed (unpaid post-winter acquisitions; card money missing from the books; a Moment invalidated by AI results recorded after the pause; a failed favorite sale still applying penalties; trust never moving; readout ties hiding causes; the yearly Ashworth loan).

## Decisions made during the build (and why)
- Sunday ball: no legalization date is sourced, so until the `sunday_ball` flag is won (the hearing card) the Reds' Sunday home dates are played at Rocky Point (1.6x draw, 40% to the resort), as Providence really did for 22 years. No year appears in any card.
- Economy at 1918 scale lands near break-even by design (spec section 8): headless 1918-1919 nets about -$1,700 to +$5,500 a year at 1,400-1,650 a game. Constants in `src/franchise/economy.ts`; M4's bots retune them.
- Gus's trust starts at 65 and the tiers are 60/50/40/25 (work / quiet / "oh meu deus" / "cabeca de mula" / "ah que porra"), so the baseline reads warm; trust moves through promises (the Sunday petition is a community promise kept by the hearing) and exploitation (cutting the men's pay).
- Within a readout place the causal (flag) entries precede the band entries, because a tie on specificity goes to file order and causes must beat bands.
- The Moment's pitch model is calibrated with oracle read policies (a typical 70/30 read within 0.06 on-base and 0.05 K of the engine; a perfect read gains at most 0.10; always swing or always take lose). The plan's coin-flip policy was unsatisfiable and was replaced.
- Holdouts are not in M3: a cut deeper than 20% is refused, never held out.

## Open threads / next steps
1. **Alex: browser click-through of M3** (`npm run dev` in the game repo). Check: the winter office (letter, sign a contract, cut a local man, set a price, fund a repair, borrow), the winter card, Open the season, the opening card, Play ball, the Opening Day story game (Manage one, Sim the other year), Sim to the next event through the block cards, a card you cannot afford shows its reason, the fall books, the walk reads differently after different choices, 1919's letter changes tone with the result, and a Moment if one fires (late game, Gus up, race alive). Reload mid-card and mid-Moment.
2. **Alex: name the six placeholder clubs** in `src/league/cities.ts`.
3. **M4 plan** (three seasons 1918-1920 with the 1920 shift; the strategy bots over 1918-1930 with placeholder content; the 1929-30 money stress test; go/no-go gate). Carry-overs for M4 in priority order: holdouts; rest days between starts (the K leader problem); position-specific defense and the out-of-position penalty; prices revealing hidden potential (value() blends potential under 27); AI clubs' own books (they get a flat net by tier); other cities' Sunday laws; the winter deck is thin after 1920 (three winter cards, all one-time); the three verdicts and the last walk of 1930 (M5); the empty `clubhouse`/`bleachers` readout places; W/L decisions, injuries, double switches, hit-and-run, file export.
4. Memory: `project_hometown_rafters.md`.

## Previous: M2 (league) BUILT and pushed (2026-10-20)

## M2 (league) BUILT and pushed
- Game repo `D:\CodexWorkspaces\hometown-rafters-game` (private, github walshja9/hometown-rafters-game), main at 9748994, 114 tests green, build clean. 24 commits on top of M1's 31c5e4b. Built by opus subagents (Alex's call) with a task-gate review per batch and a final whole-branch review; the run ledger is in the game repo at `.git/sdd/progress.md`.
- What M2 is: eight generated clubs, 154-game schedule with dates, day-by-day season, standings/stats/leaders, trades/sales/purchases through one valuation with a 15% spread, pool of independent clubs, offseason development/retirement/rollover, save v2 (whole league, with an in-app "Start a new league" boundary for a broken save), League/Clubs/Deals/Game screens; the Reds' games play one side on the diamond, the AI manager runs the other.
- Plan bugs found in execution are corrected in the plan master copy (`docs/superpowers/plans/2026-10-19-m2-league.md`, "Execution notes" at the end).

## Decisions made during the build (and why)
- Per-pitch **Take is hidden** on the Game page: the chain model's per-pitch odds ignore the count, so taking until two strikes roughly doubled scoring (measured 9.6 vs 4.0 R/G in 1918). The engine keeps the action for Moments, which need a count-aware model first (M3).
- The user's club is refilled each offseason only to its **pre-retirement size** (legality first). Refilling to 25 opened a sell-and-refill money loop (+30–40 wins); refilling only to legality starved the Reds to 13 men by 1926. M3's budget and Roots signing replace this rule.
- `rotationStale` is one shared rule (game day and offseason); the Club page swaps rotation slots so one ace cannot start every game.
- AI manager bug fixed: it pinch-hit for its own pinch hitter in the P slot (172 → 62 PH subs per 40 games). Calibration still passes.

## Open threads / next steps
1. **Alex: browser click-through of M2** (`npm run dev` in the game repo). No Chrome extension was available, so only SSR smoke tests and logic probes ran. Check: standings; Play next game → diamond, one side; "Pitch to him" advances a plate appearance when fielding; Back to the league closes the day; Clubs lineup/rotation editors; Deals refuses a fleece with a reason; reload keeps state.
2. **Alex: name the six placeholder clubs** in `src/league/cities.ts` (Boston Harbormen, Buffalo Lakers, Pittsburgh Forgemen, Cleveland Lakeshores, Cincinnati Flying Pigs, Baltimore Canners) before M3 content.
3. **M3 plan** (one complete 1918: budget letter, roster moves, cards, two story games, a Moment, settlement, ballpark walk). Carry-overs that M3 must take on, in priority order: the spec's "buyer must need the player" check on sales (money exploit once income exists); Sunday home dates are very uneven per club (schedule flip keyed to ring slot) and must be balanced before gate money; a multi-season calibration test (AI rosters drift ~5 rating points down by 1930; R/G ran −9 to −16% in some seeds by the mid-1920s); count-aware pitching for Moments; rest days between starts (today the best reliever can start the next day); position-specific defense with a real out-of-position penalty (M2 defense is one fielding number, so the auto lineup plays a left fielder at short when his bat is better; seen in Alex's 10/20 click-through); prices reveal hidden potential (value() blends potential for under-27s). Full minor list: the game repo ledger and README carry-overs.
4. **Alex's 10/20 full-slice click-through (1918→1930) confirmed three carry-overs with numbers:** league R/G 4.82 in 1930 vs real 5.55 (−13%, the multi-season drift); Boston won 8 of 12 pennants (rich-tier head start never erodes because AI clubs never deal with each other); star tails missing (HR leader 30 in 1930 vs real 56; K leader 282 because starters have no rest days). Game, box score, roster screens and the 1930 stop all behaved.
5. Memory: `project_hometown_rafters.md`.


## Latest (M1 reviewed, M2 planned)
- **M1 review (game repo `D:\CodexWorkspaces\hometown-rafters-game`, 59 tests green):** accepted. Three fixes carried into the M2 plan as Tasks 1–2: "take" walked 96.5% of plate appearances (zone conversion fix), a shapeless v1 save blanked the app (load falls back), and a corrupt save could be rotated over a good one.
- **M2 plan written:** `docs/superpowers/plans/2026-10-19-m2-league.md` (14 tasks, full code). Eight generated clubs, 154-game schedule, standings/stats/leaders, transactions through one valuation with a 15% spread, independent-club pool, offseason development and retirement, save v2, League/Club/Deals/Game screens. Deferred list at the top of the plan.
- **Decision pending for Alex:** six of the eight founding clubs are placeholders in the plan's `src/league/cities.ts` (Boston Harbormen, Buffalo Lakers, Pittsburgh Forgemen, Cleveland Lakeshores, Cincinnati Flying Pigs, Baltimore Canners). Rename before M3 content.
- **Next:** Alex picks the execution mode (subagent-driven with Sonnet subagents per task, or inline as M1 was built), then execute the M2 plan (Fable builds directly, no Codex; subagents pass model sonnet), then the M3 plan (one complete 1918). Plan tasks are checkboxes; tick them as they land.


## Current state
- **Repo in sync.** Local and GitHub had unrelated, diverged histories, and auto-git couldn't push after 6/12. Merged and pushed as `2cf7aec`. Upstream is now set to `origin/main`.
- **Rhode Island Reds:** fully drafted (bible plus 16 subsystem docs).
- **St. Louis Perfectos:** only `cities/st-louis-perfectos/systems/choice-tree.md` exists (Eras 1–3, Hank Keller). The franchise was renamed from "Saints".
- **Lost for good:** St. Louis `research/historical-reference.md` and Reds `systems/final-polish.md`. Neither was ever saved. Alex: "basically a fresh start from the pre-existing repo." README marks both as missing.
- **Full review done:** `REVIEW-2026-10-19.md` at repo root. About 100 findings across Reds story, Reds mechanics, Reds history and St. Louis, with a top-10 fix order. Nothing in it has been applied yet.

## Key decisions and why
- **Merge rule:** kept the local `cities/` layout plus CLAUDE.md and the St. Louis tree; took GitHub's newer bible revision and README, with paths fixed.
- **Root cause of the drift:** the bible embeds 97–100% verbatim copies of 9 subsystem docs. The 7 newer fix docs (structural-fixes-audit, fatal-flaw-fixes, longest-game, ownership-lineage, walt-callahan-broadcaster, boston-traitor-and-arruda-family, gus-almeida) were never merged back. Fixing the duplication first means every later fix happens once instead of twice.
- **Docs-only repo:** Fable writes directly. No Codex.

## UPDATE (10/19, later): game design approved and written

M1 (baseball core) is built in `D:\CodexWorkspaces\hometown-rafters-game` (private GitHub repo walshja9/hometown-rafters-game): 59 tests green, calibrated to 1918–1930 league rates (table in its README, with the M2 carry-overs). Next: the M2 plan.

The brainstorm finished. The approved design is `docs/superpowers/specs/2026-10-19-era1-playable-slice-design.md` (commit f6af4d7). It covers an Era 1 (1918–1930) slice with OOTP-style management, the hidden Gus Standard, a TypeScript/React stack and milestones M1–M5. Fable builds it directly, with no Codex. Next: Alex reviews the spec, then the M1 implementation plan (writing-plans). The section below is the earlier state, kept for history.

## NEW DIRECTION (10/19, after the review): Alex wants to turn this into a playable game
This replaces the doc-cleanup order below as the main goal. Start the next chat with the brainstorming skill.

Fable's recommendation, not yet agreed:
- **Build a vertical slice first:** the Reds' Gus eras, 1918–1959 (Eras 1–3). It's the most complete material and has the best diegetic readouts.
- **Make it a narrative management game, not a baseball sim.**
  - Each season is a handful of second-tier choice cards plus an abstracted season result (roster strength + budget + luck; no pitch-by-pitch sim).
  - A "walk the ballpark" screen shows the hidden Gus Standard only through prose details: the old-timer, coffee milk, the Salute, Gus's Portuguese.
  - Choices live in data files (JSON/YAML) read by a small engine. The writing is the hard part, and it must stay editable.
- **The thing to prove in playtesting:** does a player who never sees a number feel the Gus Standard dropping?
- **What it needs:** the review's state model (fix #4) and its "gaps that block building Era 1" list now come first: economy numbers, path/momentum/Standard state, Gus trust scale, a choice scheduler, a win/score condition. The bible-spine refactor becomes secondary.

**Open question for Alex (ask first):** what kind of game?
(A) Narrative management slice as above (recommended).
(B) Full management sim with rosters, OVR and simulated games (OOTP-lite; far bigger).
(C) Reigns-style swipe cards.
Platform (browser vs. Python desktop) comes after that.

## Doc-cleanup steps (recommended order; now secondary to the game)
1. Cut the bible down to a linking spine (premise, paths, era table, one canonical timeline, and a precedence rule: the subsystem doc wins). Deletes about 25K duplicated words.
2. Build the canonical Reds timeline (owners, legends, Bandbox, Sal/Walt, the Nephew's generations). This was the last thing offered to Alex.
3. Rewrite `final-polish.md` and build the middle path (owner columns, Slow Bleed, Compromise foil, Gus's death scene, Reluctant Farewell).
4. Define the state model: path drift scalars, the Gus Standard as 0–100 with four bands, momentum weights.
5. Close the exploits, make Burn the Benches always happen, then the history pass, then the St. Louis engine fix and history pass, then correct the README status lines.
