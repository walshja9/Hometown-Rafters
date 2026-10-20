# M3: One Complete 1918 (then 1919) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking. For M3 the implementer is Codex (one dispatch per task, write-capable, cwd = the game repo); Fable reviews each diff and writes the content in Task 10.

**Goal:** The Reds play one whole season as a franchise, not a roster: the Coalition's budget letter, contracts, prices and repairs in winter, four choice cards, two story games, an elimination Moment, the fall settlement and the ballpark walk, then 1919 begins the same way, with every hidden value moving and read back only through prose.

**Architecture:** Two new pure-TypeScript parts beside `baseball` and `league`: `franchise` (money, hidden values, drift, obligations, the phase machine) and `story` (content loading and checking, the scheduler, effects, readouts, Moments). Content is Markdown with YAML frontmatter under `content/`, loaded by Vite's `import.meta.glob` so it is data, not code. `LeagueState` grows a `phase`, a `franchise` block and a `story` block; the save moves to version 3 (v2 is refused, nothing to migrate). The React screens stay plain tables and paragraphs.

**Tech Stack:** TypeScript 7, Vite 8, React 19, Vitest 5; one new dependency, `yaml` (frontmatter and the readout/letter files).

**Repo:** `D:/CodexWorkspaces/hometown-rafters-game` (main; 114 tests green at 3e4a5c2). Spec: docs repo `docs/superpowers/specs/2026-10-19-era1-playable-slice-design.md` §4, §5, §8, §9 (Moments), §10, §11. Project rules: 65 = average position player; ratings 0–99; seeded RNG streams; viewing a screen never changes results.

## Global Constraints

- Hidden values are never shown as numbers or band names anywhere in the UI. No `trust`, `culture`, `loyalty`, `infrastructure`, `drift`, `standard` or band on screen, in a title, or in a dev-only toggle that ships. Tests read them from state.
- Money is visible: cash, debt, the season's books, prices, attendance.
- Spec §5 starting values and moves are law: trust 65 (promise kept +2, broken −5, deliberate exploitation −12; losses never move it); culture 50 (+1 per qualifying event capped at +4 a year, neglect −3, serious betrayal −8); Standard = 0.4·trust + 0.6·culture, bands 0–24 / 25–49 / 50–74 / 75–100, readout flips only after crossing a band edge by 3; loyalty 50 (affordable admission +2 a year, exclusionary −2 a year, broken community promise −6, season result ±1 at most, it drives attendance only); infrastructure 65 (skipped maintenance −8 a year, one repair unit +10, below 40 sections close); drift roots/ambition/money start 0, ×0.85 each winter, normal decision +6, landmark +12, momentum m = (leader − runner-up)/100, points toward a non-leading path are divided by (1 + m), all clamped 0–100.
- Culture never changes batting or pitching outcomes (spec §5 honesty rule). No hidden value touches the engine, the matchups or the AI manager.
- The sim never rigs games. Moments fire only from a real situation (spec §4 step 5); scores are never manufactured.
- Gus (`id: 'gus'`) is authored, `untouchable`, `local`, and plays shortstop; his statistics come from his ratings and are never forced.
- Every card has at least one option with no `spend` (the checker enforces it). Options the Reds cannot afford are disabled, never hidden.
- Determinism: the same seed and the same choices give the same state. Story draws use a new `rng.story` stream; nothing else draws from it; the league and development streams are untouched by M3 (the existing "a season never draws from the league stream" test must still pass).
- Save version 3. Bump `SAVE_VERSION` again on any later shape change.
- No new UI libraries. No Web Worker unless a season sim stalls the page (it ran in 0.8 s in M2).
- Tests: `npm test` green and `npm run build` clean at the end of every task. Commits with explicit paths, never `git add -A`; trailer `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`.
- M2 carry-overs that M3 absorbs, kept small: the "buyer must need the player" sale check (Task 3), Sunday home dates balanced per club (Task 2), the multi-season rating drift (Task 3), count-aware pitching for Moments only (Task 7). Rest days, position-specific defense, holdouts, prices revealing potential, W/L decisions, injuries stay deferred to M4.

## Economic anchors (research, 10/20)

Used for the constants below; M4's bots retune them.

- Attendance 1918 (war-shortened year): Boston Braves 84,938 total, 1,370 a game; Red Sox 3,961; Yankees 4,586. 1919: Giants 10,127, Yankees 8,909. A poor club drew about 1,400 a game, an average one 2,500 to 4,000, a rich one 8,000 and up.
- Prices: bleachers 25¢ (1918 to 1923), 55¢ by 1925; grandstand about $1 in 1923 with tax. Older receipts show 20¢ bleachers and 30¢ pavilion with the visiting club's share itemised.
- Salary: about $5,000 average in 1920 (spec §3); 1918 lower. The valuation's `BASE_VALUE` of $6,000 is the transfer value of a 65 player in his prime. Salary is half of a value computed from today's rating only (no age factor), so a 65 player earns about $3,000 in 1918.
- Sunday ball in Rhode Island: no legalization date is sourced. Providence's professional club played all its Sunday games at Rocky Point in Warwick from 1896 for 22 years because the local police did not enforce the Sunday law there. The slice uses that: until the `sunday_ball` flag is set, the Reds' Sunday home dates are played at Rocky Point (a bigger crowd, but the resort takes 40% of the gate); with the flag, Sunday games are at the Bandbox at the full Sunday draw. No year is stated in any card.

## File map

Create:
- `src/franchise/types.ts`: `Franchise`, `Hidden`, `Books`, `Loan`, `Phase`.
- `src/franchise/economy.ts`: constants, `salaryFor`, `attendanceFor`, `gameMoney`, `settle`, loans, foreclosure.
- `src/franchise/hidden.ts`: `adjust`, drift momentum, `standard`, `band`, `shownBand` (hysteresis), yearly loyalty and infrastructure moves, `gusTier`.
- `src/franchise/loop.ts`: the phase machine: `toOpening`, `advance` (sim to the next event), `toFall`, `toWalk`, `toWinter`; story-game selection; block boundaries.
- `src/story/types.ts`: `Card`, `Option`, `Effect`, `Readout`, `MomentSpec`, `StoryState`.
- `src/story/content.ts`: loads `content/**` with `import.meta.glob`, parses frontmatter, exposes `CARDS`, `READOUTS`, `MOMENTS`, `LETTERS`, `STORY_GAMES`.
- `src/story/check.ts`: `checkContent()`; plain-English errors; a test runs it over the shipped content.
- `src/story/scheduler.ts`: `deal(L, slot)` (fixed, then due, then weighted); `choose(L, optionId)`; `applyEffect`.
- `src/story/readouts.ts`: `readout(L, place)` selection by specificity.
- `src/story/moment.ts`: trigger, zone model, per-pitch resolution for Moments.
- `src/ui/OfficePage.tsx` (winter), `src/ui/CardPage.tsx`, `src/ui/StoryGamePage.tsx`, `src/ui/MomentPage.tsx`, `src/ui/WalkPage.tsx`, `src/ui/BooksPage.tsx`.
- `content/cards/*.md`, `content/readouts.yaml`, `content/moments/elimination.md`, `content/story-games.yaml`, `content/letters.yaml` (Task 10, Fable).
- `tests/franchise/*.test.ts`, `tests/story/*.test.ts`, `tests/loop.test.ts`.

Modify:
- `src/league/types.ts` (version 3, `phase`, `franchise`, `story`, `rng.story`, `salary` and `local` on players), `src/league/league.ts` (Gus, salaries, winter start, AI replacement mean), `src/league/schedule.ts` (Sunday balance), `src/league/season.ts` (per-game money into the books; `simUserGame` with the Moment check), `src/league/transactions.ts` (need check), `src/baseball/game.ts` (`applyPitchResult` split out), `src/save/save.ts` (v3), `src/ui/App.tsx` (phase routing), `src/ui/LeaguePage.tsx` (phase-aware controls, the Gus line, attendance), `README.md`.

## Shared interfaces (every task uses these names exactly)

```ts
// src/franchise/types.ts
export type Phase = 'winter' | 'opening' | 'season' | 'fall' | 'walk' | 'over';
export type Path = 'roots' | 'ambition' | 'money';
export interface Hidden {
  trust: number; culture: number; loyalty: number; infrastructure: number;
  drift: Record<Path, number>;
  cultureEvents: number;        // qualifying +1 events this year (cap 4), reset in winter
  bandShown: 0 | 1 | 2 | 3;     // last readout band, for hysteresis
}
export interface Loan { principal: number; year: number }           // outstanding principal, year taken
export interface Books {                                              // one season's money, in whole dollars
  year: number; home: number; away: number; attendance: number;      // games at home, games away, total home attendance
  gate: number; roadShare: number; resortCut: number; concessions: number; advertising: number; coalition: number; sales: number;
  payroll: number; travel: number; dues: number; upkeep: number; visitorsShare: number; interest: number; principal: number; purchases: number;
}
export interface Franchise {
  hidden: Hidden;
  price: number;                 // cents, one of PRICES
  priceHistory: number[];        // price each year, index = year - 1918
  repairs: number;               // repair units funded this winter
  concessionDeal: { until: number; perHead: number } | null;   // a Money-path contract: lump sum paid, no per-head income until `until`
  loans: Loan[];
  books: Books;                  // the running season
  lastBooks: Books | null;       // last season, for the letter and the walk
  support: number;               // Coalition support promised for this year (the letter)
  salaries: Record<string, number>; // what each Reds player signed for this year
}

// src/story/types.ts
export type Slot = 'winter' | 'opening' | 'block1' | 'block2';
export type CardPhase = 'winter' | 'opening' | 'block';
export type HiddenKey = 'trust' | 'culture' | 'loyalty' | 'infrastructure' | 'drift.roots' | 'drift.ambition' | 'drift.money';
export type Effect =
  | { spend: number } | { earn: number }
  | { adjust: HiddenKey; by: number }
  | { set_flag: string } | { clear_flag: string }
  | { followup: string; in_seasons: number }
  | { promise: { id: string; due_in: number; kept_if_flag: string; community?: true } }   // community: breaking it also costs loyalty −6
  | { landmark: Path }                                   // +12 toward a path (a normal option gives +6 via `path`)
  | { sign: { name: string; pos: Position; age: number; mean: number } }   // a local amateur joins the Reds (pool if the roster is full)
  | { sell_favorite: true }                              // the best Reds hitter other than Gus is sold to the richest AI club at askPrice
  | { concession_deal: { years: number; lump: number } }
  | { exploit: true };                                   // deliberate exploitation: trust −12 (spec §5)
export interface Option { id: string; text: string; path?: Path; effects: Effect[]; result?: string; requires?: { cash?: number } }
export interface Requires { flag?: string[]; not_flag?: string[]; path?: Path; min_cash?: number }
export interface Card {
  id: string; years: [number, number]; phase: CardPhase; fixed?: boolean; priority: number; weight?: number;
  repeat: 'never' | 'yearly'; requires?: Requires; options: Option[]; body: string;
}
export interface Readout {
  place: 'gate' | 'bleachers' | 'concession' | 'clubhouse' | 'gus';
  years?: [number, number];
  when?: { band?: 0 | 1 | 2 | 3; loyalty_below?: number; loyalty_above?: number; trust_below?: number; trust_above?: number; infra_below?: number; flag?: string; not_flag?: string; priced_up?: true; lost_pennant_race?: true };
  text: string;
}
export interface MomentSpec { id: string; years: [number, number]; trigger: 'elimination'; batter: 'gus'; setup: string; hooks: Partial<Record<'ball' | 'strike' | 'foul' | 'inPlay', string>>; aftermath: Record<'K' | 'BB' | 'out' | 'hit' | 'run', string> }
export interface Obligation { id: string; due: number; kept_if_flag: string; community: boolean }
export interface StoryState {
  flags: Record<string, true>;
  obligations: Obligation[];
  followups: { id: string; year: number }[];
  choices: { year: number; card: string; option: string }[];
  dealt: string[];                              // slot keys already filled, `${year}:${slot}`
  pending: string | null;                       // a card waiting for a choice
  storyGame: { day: number; kind: 'opening' | 'decider' | 'closing' } | null;   // waiting for Manage / Sim
  moment: { id: string; paStart: number } | null;   // live Moment: index into L.live.events where the at-bat began
  momentDone: { year: number } | null;          // one Moment a year
  gusLine: string;                              // the latest Gus readout, shown on the League page
}

// src/league/types.ts additions
export interface LeaguePlayer extends Player { pos; age; potential; untouchable?: true; local?: true; salary: number }
export interface LeagueState { version: 3; phase: Phase; franchise: Franchise; story: StoryState; rng: { league; development; story: RngState }; ... }
```

Constants, all in `src/franchise/economy.ts`, exported so tests can read them:

```ts
export const PRICES = [25, 35, 50, 75] as const;            // cents; 35 is the default
export const AFFORDABLE = 25, EXCLUSIONARY = 75;             // loyalty +2 / −2 a year (spec §5)
export const BASE_DRAW = { 1: 1400, 2: 2600, 3: 4000 };     // weekday draw by tier at 1918 scale
export const SUNDAY_HOME = 2.2;                              // Bandbox Sundays with the sunday_ball flag
export const ROCKY_POINT = { draw: 1.6, cut: 0.4 };          // Sundays without the flag: played at the resort, 40% of the gate stays there
export const AI_SUNDAY = 1.8;                                // other clubs' Sundays (their laws are not modelled)
export const ELASTICITY = 0.7;                               // demand × (35 / price) ^ ELASTICITY: raising prices raises the gate, costs fans
export const RESULTS_SWING = 0.15;                           // ±15% at most from the current season's winning pct
export const CLOSED_CAPACITY = 0.7;                          // usable capacity when infrastructure < 40
export const VISITOR_SHARE = 0.2;
export const CONCESSION_PER_HEAD = 5;                        // cents
export const ADVERTISING = 600, DUES = 500, TRAVEL_PER_ROAD_GAME = 40;
export const COALITION_SUPPORT = 2000, COALITION_LAST_YEAR = 1928;   // spec §8: support ends in 1929
export const REPAIR_UNIT = 500, REPAIR_GAIN = 10, NEGLECT = 8;
export const LOAN_CAP = 10000, LOAN_RATE = 0.08, LOAN_TERM = 3;
export const AI_NET = { 1: 2000, 2: 4000, 3: 8000 };        // ponytail: AI clubs' season net by tier; their books are not kept
export const SALARY_BASE = 3000;                             // a 65 player in 1918; salaryFor = max(600, round100(SALARY_BASE × e^(0.08·(overall−65))))
export const MAX_CUT = 0.2;                                  // a player refuses an offer more than 20% under his expectation (holdouts: M4)
export const DEMAND_GROWTH = 0.06;                           // per year 1919–1928; 1929 and 1930 each ×0.85 of the year before (spec §8)
```

Attendance for one home game (home club `h`, date `d`):

```
draw   = BASE_DRAW[tier(h)] × growth(year)
factor = price × loyalty × results × sunday
  price   = (35 / p) ^ ELASTICITY                          (Reds only; AI price is 35)
  loyalty = 1 + 0.01 × (loyalty − 50)                      (Reds only)
  results = clamp(1 + (pct − .5), 1 − RESULTS_SWING, 1 + RESULTS_SWING), pct = .5 until 10 games played
  sunday  = Reds: flag ? SUNDAY_HOME : ROCKY_POINT.draw; AI: AI_SUNDAY; weekdays 1
usable = capacity × (Reds && infrastructure < 40 ? CLOSED_CAPACITY : 1)
attendance = min(usable, round(draw × factor))
gate = attendance × price / 100 (dollars); visitor gets VISITOR_SHARE of it; a Rocky Point Sunday loses ROCKY_POINT.cut of the home share first.
```

Only the Reds' games are booked (home: gate net of visitor share and the resort cut, concessions; away: VISITOR_SHARE of the AI home gate). AI clubs get `AI_NET[tier]` at settlement and nothing else.

## Phase machine (`src/franchise/loop.ts`)

```
winter  → OfficePage. Letter, contracts, price, repairs, loans, deals, the winter card (slot 'winter'). "Open the season" → toOpening.
opening → lineup check, the opening card (slot 'opening'). "Play ball" → phase 'season', storyGame = { day 0, 'opening' }.
season  → LeaguePage. advance(L) sims Reds games until: a pending card, a block boundary not yet dealt (day 52 → 'block1', day 104 → 'block2'),
          a story game (day 0 opening; the decider: the first day on which a Reds loss would end the race while the Reds are within
          the games remaining of the lead with ≤ 10 games left; else the last home game as 'closing'), a Moment, or day 154.
          Play next game / Sim day / Sim week keep working but stop at the same events.
fall    → BooksPage. settle(L) ran on entry; shows the books, loans, foreclosure if any. "Walk the ballpark" → toWalk.
walk    → WalkPage: three scenes (gate, concession, gus). "Next year" → toWinter: offseason(L) (development, retirement, AI refill),
          drift ×0.85, cultureEvents 0, price history pushed, new letter, phase 'winter' (or 'over' after 1930 / foreclosure).
over    → the end screen (foreclosure headline, or "The slice ends in 1930." with the three verdicts deferred to M5).
```

`advance` never plays an AI-only day past a Reds event, and every stop is persisted (the save is written by App's `run` as today).

## Content formats (Task 10 writes to these; Task 5 reads them)

Card file `content/cards/<id>.md`:

```markdown
---
id: sunday-rocky-point
years: [1918, 1919]
phase: winter
priority: 80
repeat: never
requires: { not_flag: [sunday_ball] }
options:
  - id: petition
    text: Back the junior partners' petition to play Sundays at the Bandbox.
    path: roots
    effects:
      - { spend: 300 }
      - { adjust: culture, by: 1 }
      - { followup: sunday-hearing, in_seasons: 1 }
    result: The bakery owner carries the petition to the State House himself.
  - id: defer
    text: Keep the Rocky Point arrangement another year.
    path: money
    effects:
      - { adjust: loyalty, by: -2 }
---
Body prose, Markdown, one to three paragraphs.
```

`content/readouts.yaml`: a list of `Readout`. Every `place` needs one entry with no `when` and no `years` (the fallback). `content/letters.yaml`: `{ year: number, support: number, text: string }[]` with `{cash}`, `{net}`, `{support}` placeholders and a `tone` per entry (`'even' | 'warm' | 'cold'`) chosen by last year's net (≥ 0 even/warm; < −1000 cold). `content/story-games.yaml`: `{ opening: { title, text }, decider: { title, text }, closing: { title, text } }`. `content/moments/elimination.md`: frontmatter `MomentSpec` minus `setup`, the body is `setup`.

---

### Task 1: Save v3, franchise and story state, Gus, salaries

**Files:**
- Create: `src/franchise/types.ts`, `src/story/types.ts`, `src/franchise/economy.ts` (constants and `salaryFor` only), `src/franchise/hidden.ts` (`HIDDEN_START` only)
- Modify: `src/league/types.ts`, `src/league/league.ts`, `src/save/save.ts`, `src/ui/App.tsx` (type only), `tests/league/helpers.ts`, `tests/save.test.ts`
- Test: `tests/franchise/state.test.ts`

**Interfaces:** Produces every type in "Shared interfaces"; `newLeague(seed)` returns `phase: 'winter'`, `franchise`, `story`, `rng.story`, Gus on the Reds, `salary` on every player; `salaryFor(p)`; `HIDDEN_START`.

- [ ] Add the types above verbatim. `LeagueState.version` becomes `3`; `SAVE_VERSION = 3`.
- [ ] `salaryFor(p) = Math.max(600, Math.round(SALARY_BASE * Math.exp(0.08 * (overall(p) - 65)) / 100) * 100)`. `genPlayer` sets `salary: salaryFor(p)` after potential.
- [ ] Gus: in `newLeague`, after `genRoster` for `RI`, replace the generated starting shortstop (the `SS` hitter with the best `batValue`) with `{ id: 'gus', name: 'Gus Almeida', bats: 'R', throws: 'R', pos: 'SS', age: 18, batting: { contact: 72, power: 45, eye: 68, avoidK: 78, speed: 85, fielding: 88, arm: 75 }, potential: 82, untouchable: true, local: true, salary: salaryFor(...) }`. `seq` is not consumed. The replaced man goes to the pool at `askPrice(value)`.
- [ ] `newLeague` initialises `phase: 'winter'`, `franchise: { hidden: HIDDEN_START (trust 65, culture 50, loyalty 50, infrastructure 65, drift all 0, cultureEvents 0, bandShown: 2), price: 35, priceHistory: [], repairs: 0, concessionDeal: null, loans: [], books: emptyBooks(year), lastBooks: null, support: COALITION_SUPPORT, salaries: {} }`, `story: { flags: {}, obligations: [], followups: [], choices: [], dealt: [], pending: null, storyGame: null, moment: null, momentDone: null, gusLine: '' }`, `rng.story = seedState(`${seed}:story`)`.
- [ ] `helpers.ts bare()` gains the same fields. Fix every test that builds a `LeagueState` literal.
- [ ] Tests: a new league is in winter with Gus at short on the Reds (`untouchable`, `local`, age 18); every player has a salary ≥ 600; `salaryFor` of a 65 player is 3000 and of an 85 player about 14,900; v2 JSON is refused by `deserialize`; `npm test` green.
- [ ] Commit: `feat(m3): save v3, franchise and story state, Gus, salaries`.

### Task 2: Economy, per-game books, settlement, loans, foreclosure, Sunday balance

**Files:**
- Modify: `src/franchise/economy.ts`, `src/league/season.ts`, `src/league/schedule.ts`, `src/league/types.ts` (`Result.attendance`)
- Test: `tests/franchise/economy.test.ts`, `tests/league/schedule.test.ts`

**Interfaces:**
- `attendanceFor(L, home: string, date: string): number`; `gameMoney(L, g: Game): { attendance: number; homeNet: number; visitorShare: number; resortCut: number; concessions: number }` (dollars, whole).
- `record()` in season.ts books the Reds' side of every Reds game into `L.franchise.books` (home: gate, resortCut, concessions, visitorsShare, attendance, home++; away: roadShare, travel += TRAVEL_PER_ROAD_GAME, away++) and stores `attendance` on the `Result`.
- `settle(L): { foreclosed: boolean }` at season end: `payroll = Σ salaries`, `dues`, `upkeep = repairs × REPAIR_UNIT` (already paid in winter: book it, do not charge twice), `advertising`, `coalition = year ≤ COALITION_LAST_YEAR ? support : 0`, loan `interest` and `principal` (each loan pays `principal/LOAN_TERM` + `LOAN_RATE × outstanding`, removed at 0), `sales` and `purchases` from the year's transactions (hook `me.cash` deltas in transactions.ts: add `L.franchise.books.sales += price` / `purchases += price`). Cash = cash + income − expenses. AI clubs: `cash += AI_NET[tier]`.
- Foreclosure: if cash < 0 after settlement, borrow up to `LOAN_CAP − outstanding` (a new loan); still < 0: sell Reds players by descending `value`, skipping `untouchable`, to the richest AI club at `bidPrice` while the roster stays legal, until cash ≥ 0; still < 0: `foreclosed: true` and `L.phase = 'over'`, `L.story.flags.foreclosed`.
- `borrow(L, amount)`: refuses over the cap or during the season; `repay(L, amount)` optional, not required.
- Schedule: after building games, balance Sunday home dates: repeat until the spread of per-club Sunday home counts is ≤ 2 or no swap helps: for each Sunday game whose home club has the most Sunday homes and whose away club has the fewest, swap home/away on it and on one non-Sunday game of the same pair with the opposite hosting (so the pair stays 11/11). Deterministic (iterate in day order).

- [ ] Tests: attendance of a weekday Reds game at 35¢ with neutral values equals `BASE_DRAW[1]` (1400); 25¢ raises attendance and 75¢ lowers it; a Rocky Point Sunday draws 1.6× and the Reds keep 60% of the home share; with `sunday_ball` set it draws 2.2× with no cut; attendance never exceeds capacity, and 70% of it when infrastructure is 39; a full simmed 1918 books 77 home and 77 away games, gate > 0, and `cash_after = cash_before + Σincome − Σexpenses` to the dollar; a loan of 9,000 pays 3,000 principal + 720 interest the first fall; `borrow` over the cap is refused; a Reds club with cash −50,000 and nothing to sell forecloses (`phase 'over'`); Sunday home counts across the eight clubs differ by at most 2 and every pair still splits 11/11; `rng.league` is untouched by a season.
- [ ] Commit: `feat(m3): gate, books, settlement, loans, foreclosure; balanced Sundays`.

### Task 3: Winter office rules: letter, contracts, price, repairs; sale need check; replacement drift

**Files:**
- Modify: `src/franchise/economy.ts` (`renew`, `setPrice`, `fundRepairs`), `src/league/transactions.ts` (need check), `src/league/league.ts` (`fillRoster` AI mean 60/58; the Reds refilled to legality only, since income now exists), `src/story/content.ts` stub for `LETTERS` is NOT needed here: the letter text comes in Task 5; this task produces the numbers.
- Test: `tests/franchise/office.test.ts`, `tests/league/transactions.test.ts`, `tests/league/league-calibration.test.ts`

**Interfaces:**
- `letter(L): { support: number; lastNet: number; cash: number; tone: 'even' | 'warm' | 'cold' }`: support = `COALITION_SUPPORT` through 1928 else 0; tone from `lastBooks` net (null → even; ≥ 0 warm when the Reds finished above .500, else even; < −1000 cold).
- `renew(L, id, offer): Verdict`: `expect = salaryFor(p)`; offer ≥ expect → signed at offer (capped at 2× expect); offer < expect × (1 − MAX_CUT) → `{ ok: false, reason: 'He will not sign for that.' }`; a cut on a `local` player costs culture −1 (once a year per player). Unsigned players are signed at `expect` by `toOpening` (nobody plays unsigned). `salaries[id]` holds the signed figure; `payroll` sums it.
- `setPrice(L, cents)`: only `PRICES`; only in winter. `fundRepairs(L, units)`: charges `units × REPAIR_UNIT` now, `repairs += units`, infrastructure `+REPAIR_GAIN` per unit applied at settlement (so the walk reads it), refuses when cash is short.
- Sale need check (spec §8 "buyers must need the player"): in `sellPlayer`, the AI club must need him: a hitter must out-rate (`overall`) its current best man at `p.pos`; a pitcher must out-rate its worst rotation man. Otherwise `{ ok: false, reason: `${club} have no need of him` }`. Same check in `proposeTrade` for every player the AI receives.
- Replacement drift (M2 carry-over): AI `fillRoster` means 60 (legality) and 58 (top-up) instead of 50/55, so the league does not sink. Calibration test: `simSeason` 1918→1930 headless with `offseason` (no story), and in 1930 the league-wide mean `overall` of lineup starters is within 3 points of 1918's, and R/G is within ±10% of the 1930 era rate (the existing per-season calibration test stays).

- [ ] Tests for each verdict above, the letter tones, the need check (a 50-rated catcher is refused by a club whose catcher is 70; a 75 is taken), `fundRepairs` refusing without cash, and the 13-season drift test.
- [ ] Commit: `feat(m3): winter office rules, sale need check, replacement drift fix`.

### Task 4: Hidden values, drift momentum, the Standard, Gus tier

**Files:**
- Modify: `src/franchise/hidden.ts`
- Test: `tests/franchise/hidden.test.ts`

**Interfaces:**
- `adjust(h: Hidden, key: HiddenKey, by: number)`: clamps 0–100; `culture` positive moves count against `cultureEvents` (cap 4 a year; the 5th +1 is dropped); drift keys go through momentum: `lead = max drift`, `runnerUp = second`, `m = (lead − runnerUp) / 100`; a positive `by` toward a path that is not the leader is divided by `(1 + m)`.
- `standard(h) = 0.4·trust + 0.6·culture`; `band(x)`: 0–24 → 0, 25–49 → 1, 50–74 → 2, 75–100 → 3; `shownBand(h)`: returns `bandShown` unless `standard` has crossed a band edge by ≥ 3 past `bandShown`'s range, then updates and returns the new band.
- `leadingPath(h): Path | null` (null when all 0; ties → the most recent substantive choice, passed in as `last: Path | null`).
- `yearly(L)` at settlement: loyalty `+2` if price ≤ AFFORDABLE, `−2` if ≥ EXCLUSIONARY; `±1` by season result (above .500 +1, below −1); infrastructure `−NEGLECT` if `repairs === 0` else `+REPAIR_GAIN × repairs`; `winter(L)`: drift ×0.85 (rounded to 1 decimal), `cultureEvents = 0`, `repairs = 0`.
- `gusTier(h): 'work' | 'quiet' | 'meu_deus' | 'mula' | 'porra'` from trust: ≥ 70, 55–69, 45–54, 30–44, < 30 (spec §5 and `gus-almeida.md` escalation).

- [ ] Tests: each move; momentum (roots 30, ambition 0: +6 toward ambition lands 6/1.3 ≈ 4.6); culture cap; hysteresis (standard 74 → 76 keeps band 2; 78 flips to 3; 76 keeps 3; 71 flips to 2); gusTier edges; `winter` decay.
- [ ] Commit: `feat(m3): hidden values, drift momentum, standard bands with hysteresis`.

### Task 5: Content loading, checker, scheduler, effects

**Files:**
- Create: `src/story/content.ts`, `src/story/check.ts`, `src/story/scheduler.ts`, `content/cards/_example.md` (a fixture card that the checker test uses; deleted in Task 10), `content/readouts.yaml` (fallback entries only), `content/letters.yaml` (one 1918 entry), `content/story-games.yaml` (three titles with one-line text), `content/moments/elimination.md` (minimal valid file)
- Modify: `package.json` (`yaml`), `src/league/types.ts` if needed
- Test: `tests/story/content.test.ts`, `tests/story/scheduler.test.ts`

**Interfaces:**
- `content.ts`: `const raw = import.meta.glob('/content/cards/*.md', { query: '?raw', import: 'default', eager: true })` and the same for `moments`; YAML files via `?raw` too; `parseCard(text, file): Card` splits `---` frontmatter with `yaml.parse`. Exports `CARDS: Card[]`, `READOUTS: Readout[]`, `MOMENTS: MomentSpec[]`, `LETTERS`, `STORY_GAMES`. Vitest resolves `import.meta.glob` natively.
- `check.ts`: `checkContent(cards, readouts, moments, letters, storyGames): string[]` of plain-English errors, empty when fine: duplicate id; `years` not two ascending numbers in 1918–1930; unknown phase; no options; option without `id`/`text`; an option that has no `spend`-free sibling (`card X has no option the Reds can always afford`); unknown effect key (`card X option Y uses unknown effect "adjsut"`); `adjust` with an unknown key; `followup` to a card id that does not exist; `promise.kept_if_flag` never set by any option in any card (`card X promises "fix_grandstand" but no card sets that flag`); `requires.flag` never set anywhere; a readout `place` with no fallback; a letter missing for 1918; a Moment whose aftermath lacks a key. A test asserts `checkContent(CARDS, ...)` is `[]` over the shipped content.
- `scheduler.ts`:
  - `eligible(L, card, slot)`: year in range; `card.phase` matches the slot (`block` matches `block1`/`block2`); `repeat: 'never'` and already chosen (in `choices`) → no; `requires` all satisfied (`path` = `leadingPath`).
  - `deal(L, slot): Card | null`: slot key `${year}:${slot}` must not be in `dealt`; push it. Order: a `fixed` eligible card for this year/phase; else a due followup (`followups` with `year ≤ L.year`, by priority); else the weighted draw over eligible cards with `rng.story` (`weight ?? priority`). Sets `story.pending = card.id`. Returns null if nothing is eligible (the slot is still marked dealt).
  - `choose(L, optionId)`: applies the option's effects in order (an unaffordable option throws), `+6 drift` toward `option.path` (through momentum), logs `choices`, clears `pending`, sets `gusLine` from `readout(L, 'gus')` (Task 8; until then the tier name is fine in tests only, never in UI).
  - `applyEffect(L, e)` for every member of `Effect`: `spend` (cash, refuse if short), `earn`, `adjust`, flags, `followup` (`{ id, year: L.year + in_seasons }`), `promise` (push an `Obligation` with `due = L.year + due_in`), `landmark` (+12 via momentum), `sign` (a `genPlayer` at the given mean, `local: true`, with `rng.story`, onto the Reds if < 25 else the pool at ask 0), `sell_favorite`, `concession_deal` (cash += lump; `concessionDeal = { until: year + years, perHead: 0 }`; culture −3 as neglect of the vendors, once), `exploit` (trust −12).
  - `dueObligations(L)` at settlement: for each obligation with `due ≤ year`: kept (flag set) → trust +2; broken → trust −5 and, if `community`, loyalty −6; remove it.

- [ ] Tests: parsing the example card; every checker message above against a crafted bad card; `deal` prefers fixed over due over weighted, marks the slot, never deals twice in a slot, is deterministic by seed; `choose` applies each effect kind; an obligation kept vs broken; `sign` lands on the roster or in the pool.
- [ ] Commit: `feat(m3): content loader and checker, scheduler, effects vocabulary`.

### Task 6: The phase machine and the season flow

**Files:**
- Create: `src/franchise/loop.ts`
- Modify: `src/league/season.ts` (`simUserGame`, event stops), `src/league/league.ts` (`offseason` keeps the Reds at legality only; rotation/lineup rules unchanged)
- Test: `tests/loop.test.ts`

**Interfaces:**
- `toOpening(L)`: requires `phase 'winter'` and no `pending`; signs the unsigned at expectation; `priceHistory.push(price)`; `deal(L, 'opening')`; `phase = 'opening'`.
- `playBall(L)`: requires `'opening'` and no pending; `phase = 'season'`; `storyGame = { day: 0, kind: 'opening' }`.
- `advance(L)`: loop while `phase === 'season'`: if `pending` or `storyGame` or `live` → return; if `day === 52 && !dealt(block1)` → `deal(L, 'block1')`, return (same for 104/block2); if the Reds' game today is a story game not yet offered (`deciderDay(L)` or the last home game when no decider happened) → set `storyGame`, return; if `day >= DAYS` → `toFall(L)`, return; else `playDayStory(L)`.
- `playDayStory(L)`: like `playDay(L, true)` but the Reds' game goes through `simUserGame(L)`, which steps the engine with `aiAction` and, before each Reds plate appearance, calls `momentTrigger(L, s)` (Task 7); on a trigger it leaves `L.live = s`, sets `story.moment`, and returns without closing the day. Without a trigger the game is recorded as usual. Results are bit-identical to `simulateGame` when no Moment fires (test).
- `manageStoryGame(L)`: opens the game live (`playDay(L)` for the Reds' game only; AI games of the day sim as today). `simStoryGame(L)`: clears `storyGame` and plays the day through `playDayStory`.
- `deciderDay(L): number | null`: the first remaining day with ≤ 10 games left where the Reds are not eliminated, within the games remaining of the leader, and a loss today would eliminate them (`redsWins + remainingAfterToday < leaderWins` in the worst case); null if none.
- `toFall(L)`: `settle(L)` (Task 2) → `dueObligations` → `yearly(L)` → `phase = 'fall'` (or `'over'` on foreclosure). `toWalk(L)`: `phase = 'walk'`. `toWinter(L)`: `offseason(L)` (existing: history, development, retirement, refill, pool, schedule), `lastBooks = books`, `books = emptyBooks`, `winter(L)` decay, `support` for the new year, `deal(L, 'winter')`, `phase = year > LAST_YEAR ? 'over' : 'winter'`.

- [ ] Tests: a seeded league walked by a bot (`choose` option 0 whenever pending, `simStoryGame`, Moment resolved by swinging) goes winter 1918 → opening → season → fall → walk → winter 1919 → ... → winter 1920 with no exception, four cards dealt a year, two story games offered a year, books balanced each year, and the same seed twice gives identical JSON; `advance` stops at day 52 and 104; `deciderDay` is null for a club 30 games out and non-null for a constructed race; `playDayStory` equals `playDay(L, true)` results when no Moment fires.
- [ ] Commit: `feat(m3): phase machine, blocks, story games, sim to the next event`.

### Task 7: The Moment: trigger, count-aware pitches, calibration

**Files:**
- Create: `src/story/moment.ts`
- Modify: `src/baseball/game.ts` (split `applyAction` for `'pitch'` into `samplePitch` + `applyPitchResult(s, result, action)` so a Moment can supply the pitch result; existing behaviour and tests unchanged)
- Test: `tests/story/moment.test.ts`

**Interfaces:**
- `momentTrigger(L, s: GameState): MomentSpec | null`: `story.momentDone?.year !== L.year`; `mustWin(L)` (a Reds loss today eliminates them: `redsWins + remainingAfterToday < leaderWins`, leader ≠ Reds); `s.inning >= 7`; the Reds are batting; `currentBatter(s).id === 'gus'`; `0 ≤ (opp − reds) ≤ 1 + runnersOn`; `s.balls === 0 && s.strikes === 0`; a `MOMENTS` entry with `trigger 'elimination'` in year range.
- `ZONE_BASE = 0.55, ZONE_BALL = 0.07, ZONE_STRIKE = -0.05`: `zoneProb(balls, strikes) = clamp(ZONE_BASE + ZONE_BALL·balls + ZONE_STRIKE·strikes, 0.35, 0.8)`.
- `momentPitch(L, s, read: 'take' | 'swing' | 'protect' | 'bunt'): PitchResult`: draw `inZone` with `rng.baseball` of the game (`s.rng.baseball`, so the Moment stays inside the game's stream); `take` → in zone `strike` else `ball` (`hbp` at the base rate first); `swing` → in zone: `{ foul: IN_ZONE_FOUL, strike: whiff, inPlay: rest }`, out of zone: `{ foul: OUT_FOUL, strike: OUT_WHIFF, inPlay: OUT_INPLAY }` with in-play weights shifted to `GO`/`FO` by `OUT_WEAK`; `protect` (2 strikes only): swing with `foul` ×1.5 and in-play power weights halved; `bunt` as the engine's bunt. Then `applyPitchResult`. Constants live in one exported `MOMENT` object.
- `momentOver(L, s)`: true once the at-bat's `pa` event has been appended after `moment.paStart`; `finishMoment(L)`: `momentDone = { year }`, `moment = null`, then the rest of the game sims with `aiAction` and the day closes (`closeLive`). `aftermathKey(events)`: `'K' | 'BB' | 'out' | 'hit' | 'run'` (`run` when a run scored on the play).
- Calibration test (Codex tunes the `MOMENT` constants to pass it): for Gus vs a 65 pitcher in 1918, 20,000 Moment plate appearances under each policy: always `swing`; `take` until the first strike then `swing`; alternate; the on-base rate of every policy is within ±0.06 of the engine's simmed on-base rate for the same matchup (`simulateGame`-style PAs through `applyAction` with `normal`), and the `K` rate of "always swing" is within ±0.05 of the simmed K rate. No policy's on-base rate exceeds the simmed rate by more than 0.04.

- [ ] Tests: the trigger's each condition (one failing at a time); the pitch model respects zone probability (3-0 is more in-zone than 0-2); the calibration band; `finishMoment` closes the day with the Reds' result recorded once; a Moment never fires twice a year.
- [ ] Commit: `feat(m3): elimination Moment with a count-aware pitch model`.

### Task 8: Readouts and the ballpark walk

**Files:**
- Create: `src/story/readouts.ts`
- Test: `tests/story/readouts.test.ts`

**Interfaces:**
- `readout(L, place): string`: among `READOUTS` for the place whose `years` include `L.year` (or have none) and whose every `when` condition holds, pick the one with the most conditions (ties: file order); `band` compares to `shownBand(h)` (hysteresis; the call may update `bandShown`, so it is called from `choose`, `toWalk` and `toWinter` only, never from render); `priced_up` = price rose within the last two seasons (`priceHistory`); `lost_pennant_race` = a decider was offered and lost this year (`story.flags['lost_race_<year>']` set by Task 6 when the decider game is lost). Causal rules beat band rules by construction because they carry more conditions (spec §5 "causal readouts beat band readouts").
- `walk(L): { gate: string; concession: string; gus: string }` = the three scenes, in that order, ending on next year's problem: the `gus` scene appends one sentence from `LETTERS`-independent prose chosen by `gusTier` (the Task 10 readouts carry it; the function only selects).
- `gusLine` is `readout(L, 'gus')` after every `choose` (spec §5: Gus reacts immediately).

- [ ] Tests: fallback when nothing matches; a two-condition entry beats a one-condition entry; `priced_up` true only within two seasons of a rise; band hysteresis flows through; the walk returns three non-empty strings over the shipped content for a fresh league.
- [ ] Commit: `feat(m3): readouts by specificity and the ballpark walk`.

### Task 9: Screens

**Files:**
- Create: `src/ui/OfficePage.tsx`, `src/ui/CardPage.tsx`, `src/ui/StoryGamePage.tsx`, `src/ui/MomentPage.tsx`, `src/ui/WalkPage.tsx`, `src/ui/BooksPage.tsx`
- Modify: `src/ui/App.tsx`, `src/ui/LeaguePage.tsx`, `src/ui/GamePage.tsx` (Moment mode: pitch buttons Take / Swing / Protect / Bunt only, no other actions, no sim buttons), `src/ui/styles.css` (minimal)
- Test: `tests/ui.test.tsx` (SSR smoke per page), `tests/gamePage.test.ts`

Plain screens, period text where cheap (the letter reads as a letter; the books as a ledger). Rules:
- App routes by `L.phase` first: `winter` → Office (tabs Clubs and Deals stay available), `opening` → Office's opening section (lineup warning + Play ball), `season` → League (plus Card/StoryGame/Moment/Game overlays when `pending`/`storyGame`/`moment`/`live` are set, in that precedence), `fall` → Books, `walk` → Walk, `over` → the end screen.
- OfficePage: the letter (from `LETTERS` by year with placeholders filled, tone picked by `letter(L)`), cash, debt and loans with a Borrow control (amounts 1,000 to the cap), ticket price radio (`PRICES`), repairs (units with cost), the contracts table (name, pos, age, expectation, an offer field with Sign; signed figure shown), and the winter card if pending. "Open the season" disabled while a card is pending. No hidden number anywhere.
- CardPage: body prose, option buttons with `text` (disabled with "cannot afford" when `spend` > cash), then the chosen option's `result` and a Continue.
- StoryGamePage: title and text from `STORY_GAMES[kind]`, Manage / Sim buttons.
- MomentPage: `setup` prose, the diamond and count from GamePage's parts, the four read buttons, each pitch's `hooks` line, then the `aftermath` paragraph and "Finish the game".
- BooksPage: the ledger as two columns (income lines, expense lines), net, cash, loans outstanding, a foreclosure headline when `flags.foreclosed`; "Walk the ballpark".
- WalkPage: three paragraphs; "Next year" (or "The end" when the year is 1930).
- LeaguePage: adds "Sim to the next event" (`advance`), keeps Play next game / Sim day / Sim week (each stops at events), shows today's attendance on the last result, the Gus line as a plain sentence under the controls, and drops "Start next year" (the walk does it).
- The end screen (`over`): the foreclosure headline, or the 1930 closing line; a "New league" button.

- [ ] SSR smoke tests render each page from a fresh league at the right phase without throwing, and assert that no rendered HTML contains the strings `trust`, `culture`, `loyalty`, `infrastructure`, `drift` or `standard` (case-insensitive) for a league in each phase.
- [ ] Commit: `feat(m3): office, card, story game, Moment, books and walk screens`.

### Task 10: Content for 1918–1919 (Fable writes; Codex is not dispatched)

**Files:** `content/cards/*.md` (12 cards), `content/readouts.yaml` (~24 entries), `content/moments/elimination.md`, `content/story-games.yaml`, `content/letters.yaml` (1918–1930 support figures; prose for 1918–1920, a plain fallback after), delete `content/cards/_example.md`.

Card list (ids), all drawn from `choice-tree.md` Era 1 and the review's history fixes:
- winter 1918–19: `sunday-rocky-point` (petition / keep Rocky Point), `grandstand-repairs` (pass the hat / Ashworth's loan with strings = `promise` / let it go), `sponsor-draper` (a local businessman's money with his name on things).
- opening 1918–20: `the-rooster` (the live rooster incident: lean in / respectable), `family-at-the-gate` (a Portuguese family can't pay: let them in / protect the gate), `providence-column` (the condescending piece).
- block 1918–21: `chourico-vendor` (the Italian vendor vs "professional" concessions; the Money option is a `concession_deal`), `the-mill-kid` (the French Canadian kid; `sign` him with money to his family, or let him walk), `thin-week` (cut pay / fill seats cheaply / a sponsor's night), `founding-favorite` (`sell_favorite` for cash, or refuse; Gus has an opinion), `shift-whistle-salute` (the Seventh Inning Salute is born at the 3 PM Slater Mill whistle; fixed 1919 block1, sets `salute`), `sunday-hearing` (followup of the petition; sets `sunday_ball` on one option).
Readouts: `gate`, `concession`, `gus` with a fallback, one per band, and causal entries (`priced_up` + `loyalty_below: 45` → the 1918 family is missing; `flag: salute` → the whistle and the hats; `flag: concession_deal` → the vendor gone; `not_flag: sunday_ball` → the trolley to Rocky Point). Gus entries per `gusTier`: arrives early and says nothing / quiet / "oh meu deus" / "cabeça de mula" / "ah que porra", always about the front office, never to it (`structural-fixes-audit.md` FIX 4). No coffee milk, no parking-lot old-timer (spec §5).
- [ ] `checkContent` returns `[]`; the loop test still passes over the real content; commit `content(m3): 1918–1919 cards, readouts, Moment, letters`.

### Task 11: Docs, ledger, README, final review

- [ ] README: "Status: M3 (one season)" with what M3 is, the economic constants table and anchors, the deferred list (holdouts, rest days, position defense, prices and potential, W/L decisions, injuries, AI clubs' books, Sunday laws elsewhere, the six verdicts of 1930, bots).
- [ ] Copy the run ledger and reports to `docs/m3-build/`. Update the docs-repo HANDOFF.md and the plan's "Execution notes".
- [ ] Final whole-branch review (Fable), fix wave if needed, push.

## Execution notes

(Filled in during the build: plan bugs found in execution and how they were corrected.)
