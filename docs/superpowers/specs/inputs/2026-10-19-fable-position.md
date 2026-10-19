# Fable position (written before reading Astra's)

Decisions: keep all six. One amendment: the Moment needs a pitch-level model calibrated so its outcome distribution matches the PA model for the same matchup; the player's reads shift it. Otherwise Moments break balance.

Season loop (~15-20 min/season, ~4 h slice):
1. Winter: owner's budget letter; roster moves (trade, buy from minor-league clubs, sell for cash); contracts under the reserve clause (salary offers, holdouts); 1-3 second-tier cards. One top-tier choice in the era.
2. Spring: lineup and rotation.
3. Season: sim in chunks ("sim to next story beat"); newspaper standings between chunks; 1-3 story games per season, managed inning by inning; Moments inside them when triggered.
4. Fall: results plus the gate ledger, then Walk the Park.
5. 1930: era close. Final walk, newspaper retrospective, Gus's line, carry-forward ("how exposed are you when the Depression hits").

State: three drift scalars (Roots/Ambition/Money, 0-100) plus top-tier flags; path classified at era end; momentum = reversal costs extra choices. Gus trust 0-100 hidden, four bands (Eras 1-3; it becomes the Standard at his exit). Loyalty hidden; attendance visible. Economy in period dollars: revenue = attendance x price + concessions; payroll; owner budget. Era 1 Money options are thin by design ("the money hasn't arrived"); Sunday ball 1919 is the Era 1 fight (review). Win: no single score shown; internally score pennants, solvency and the town's verdict for balance testing. Fail state: the owner sells or moves the club.

League: 8 teams, 154 games, 25-man rosters, fictional players, 20-80 ratings (period-flavored, OOTP-familiar), hidden potential, aging.

Sim: port log5; per-season era tables (dead ball 1918-19, live ball 1920+), calibrated to real league averages. Seam: resolvePA(state, batter, pitcher, decisions, rng). Managed games add decisions between PAs; Moments resolve pitch by pitch but are calibrated to the PA distribution.

Content: Markdown + YAML frontmatter, one card per file (conditions, choices, effects), compiled to JSON and schema-validated at build. Readouts are a table: band x era x location -> prose variants. Ink (inkjs) is an option later for long dialogue.

Architecture: pure TS core with no DOM (state, seeded RNG, sim, league, economy, story scheduler, save = JSON state plus seed). UI = React + Vite. Tests (vitest): determinism (same seed, same season); calibration (10 sim seasons inside era tolerances); content validation; strategy bots (all-Roots / all-Money / middle) play 13 seasons headless and report outcomes, which is the balance test the review asked for (corruption dominance).

Risks: 1 writing volume; 2 the Gus Standard too unreadable or too readable; 3 era sim calibration; 4 OOTP feature creep; 5 the name "Rhode Island Reds" has history (the AHL Providence Reds were also called that), so check trademark before selling; 6 the review's history fixes must land in Era 1 content.

Cut if long: managed story games (keep Moments), minor-league purchases, aging detail, the Steam wrapper.

First three: (1) state model + content schema spec (Fable writes; it's the blocker); (2) headless core: RNG, ported era sim, season sim, tests; (3) ugly end-to-end loop: 3 cards, 1 readout screen, playable.
