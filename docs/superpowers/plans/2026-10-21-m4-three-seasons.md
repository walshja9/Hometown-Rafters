# M4: Three Seasons (1918–1920), the Bots and the Gate — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking. The implementer is Codex (one dispatch per task, write-capable, cwd = the game repo, verifies with `npm test -- --configLoader native --pool threads` and `npm run build -- --configLoader native`, cannot write `.git`); Fable reviews each diff, runs `npm test` and `npm run build`, commits with explicit paths, and writes the content in Task 7.

**Goal:** The Reds' first three seasons play as a franchise with the 1920 shift in the ball and in the story, the simulation stops lying in the ways Alex's click-throughs caught (starters never rest, a left fielder plays short for free, prices and ledgers are flat), and seven strategy bots play 1918–1930 headless so the numbers are tuned before M5 writes thirteen seasons of content. The go/no-go gate at the end is human: is it fun, and can a playtester read the Standard.

**Architecture:** No new subsystems. Four small truths are added to the existing engine and league (`lastStart`/tired pitchers, position-weighted defense with an out-of-position table, holdouts on the contract, per-city Sunday law); the AI clubs get real books and one winter purchase pass instead of a flat net; the season loop gains one more stop (the pennant clinch) and the story gains `won_pennant`/`pennant_last`. The bots live under `tests/bots/` as a headless runner over the phase machine, writing a Markdown report that Task 9 tunes against. Save version 6 for all of M4 (v1–5 refused, no migration).

**Tech Stack:** unchanged (TypeScript 7, Vite 8, React 19, Vitest 5, `yaml`).

**Repo:** `D:/CodexWorkspaces/hometown-rafters-game` (main at bb92ddd, 406 tests green). Spec: docs repo `docs/superpowers/specs/2026-10-19-era1-playable-slice-design.md` §6 (contracts/holdouts), §7 (rival owners), §8 (money), §9 (engine), §11 (bots), §13 (M4). Alex's decisions: no playoffs for era 1 (10/21, "playoffs are fine right now"); the pennant winner is the champion.

## Global Constraints

- Everything in the M3 plan's Global Constraints still binds: hidden values never shown as numbers or bands; the sim never rigs games; Gus (`id: 'gus'`) authored, untouchable, local, shortstop; every card has an option with no `spend`; story draws only from `rng.story`; `npm test` green and `npm run build` clean at the end of every task; commits with explicit paths, trailer `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`.
- Determinism: the same seed and the same choices give the same state. The "a season never draws from the league stream" test must still pass; AI winter dealing (Task 4) uses no RNG at all.
- Era truth: Sunday law dates are the researched ones below; no year appears in any card's prose; nothing in the engine uses a hidden value.
- Save version 6 from Task 1 on. Bump again only if a later milestone changes shape.
- No new dependencies, no Web Worker, no new UI library. Screens stay plain.
- Out of scope for M4 (M5 or later): the 1921–1930 deck proper, the three verdicts and the last walk of 1930, W/L decisions, injuries, double switches, hit-and-run, file export, AI-to-AI trades, the color line content, the period look.
- Content is Fable's (Task 7); Codex never writes card, readout or notice prose.

## Sunday law research (10/21, used by Task 5)

City-by-city, which is how it was: Cincinnati and St. Louis legal from 1902; Cleveland from 1918; Baltimore rescinded its ban in 1918 but allowed no paid admission, so paid Sunday ball is treated as 1919; New York State (Buffalo) 1919; Massachusetts voted in 1928 and Boston's first Sunday game was April 1929; Pennsylvania (Pittsburgh) not until 1933, so closed for the whole slice. Rhode Island stays as M3 built it: Rocky Point until the `sunday_ball` flag. Sources: Wikipedia "Sunday sporting events"; Library of Congress law blog, "Baseball Blues: Sunday Games and the Law" (2018); WBUR, "Sunday baseball, Red Sox, blue laws" (2018).

## File map

Create:
- `src/baseball/defense.ts`: `POSITION_WEIGHT`, `outOfPosition(natural, played)`, `defenseRating(players, lineup)`.
- `content/notices.yaml`: short stop-the-sim texts (`clinch`).
- `content/cards/the-lively-ball.md`, `content/cards/pennant-winter.md`, `content/cards/the-strike.md` (Task 7, Fable).
- `tests/bots/run.ts` (headless runner), `tests/bots/policies.ts` (the seven bots), `tests/bots/bots.test.ts` (the gate), `docs/m4-build/bots-report.md` (written by the test).
- `tests/baseball/defense.test.ts`, `tests/league/rest.test.ts`, `tests/franchise/holdout.test.ts`, `tests/franchise/ai-books.test.ts`, `tests/league/sunday-law.test.ts`, `tests/franchise/clinch.test.ts`.

Modify:
- `src/baseball/types.ts` (Player gains `pos?`, `tired?`), `src/baseball/game.ts` (defense rating, tired pitchers), `src/league/lineup.ts` (fit uses the table; holdouts excluded; `toTeam` takes the day), `src/league/season.ts` (rest bookkeeping, holdout expiry), `src/league/types.ts` (Club `lastStart`, `lastPitched`; LeaguePlayer `holdout`; City `sundayFrom`; version 6), `src/league/cities.ts`, `src/league/schedule.ts` (closed cities host no Sundays), `src/league/league.ts` (AI winter dealing; `pennant_last`), `src/franchise/economy.ts` (holdout band in `renew`; AI books in `settle`; Sunday factor by law), `src/franchise/loop.ts` (clinch stop; `pennant_${year}`), `src/story/types.ts` + `src/story/readouts.ts` (`won_pennant`), `src/story/content.ts` + `src/story/check.ts` (notices), `src/ui/App.tsx` (notice page), `src/ui/OfficePage.tsx` (holdout warning), `src/ui/ClubPage.tsx` (holdout mark), `src/save/save.ts` (v6), `tests/league/helpers.ts`, `README.md`.

---

### Task 1: Pitchers rest (starters every fourth day, tired relievers sit)

**Files:**
- Modify: `src/league/types.ts`, `src/baseball/types.ts`, `src/baseball/game.ts` (`availablePitchers`), `src/league/lineup.ts:64-70` (`toTeam`), `src/league/season.ts:22-50` (`record`), `src/league/league.ts` (`newLeague`, `offseason`), `src/save/save.ts`, `tests/league/helpers.ts`
- Test: `tests/league/rest.test.ts`

**Interfaces:**
- Produces: `Club.lastStart: Record<string, number>` (pitcher id → day of his last start), `Club.lastPitched: Record<string, { day: number; pitches: number }>`; `toTeam(club, name, day)` (third argument required); `Player.tired?: true`; `REST_DAYS = 3`, `TIRED_PITCHES = 35`, `TIRED_DAYS = 1` exported from `src/league/lineup.ts`; `LeagueState.version: 6`, `SAVE_VERSION = 6`.

- [ ] **Step 1: Write the failing tests**

```ts
// tests/league/rest.test.ts
import { expect, test } from 'vitest';
import { newLeague } from '../../src/league/league';
import { REST_DAYS, toTeam } from '../../src/league/lineup';
import { playDay } from '../../src/league/season';

test('a starter never starts again within REST_DAYS, even when the rotation is reordered', () => {
  const L = newLeague('rest');
  L.phase = 'season';
  for (let d = 0; d < 12; d++) playDay(L, true);
  const me = L.clubs.RI!;
  const yesterday = Object.entries(me.lastStart).sort((a, b) => b[1] - a[1])[0]![0];
  me.rotation = [yesterday, ...me.rotation.filter((id) => id !== yesterday)];
  me.nextStarter = 0;
  expect(toTeam(me, 'Reds', L.day).startingPitcherId).not.toBe(yesterday);
  expect(L.day - me.lastStart[yesterday]!).toBeLessThanOrEqual(REST_DAYS);
});

test('a reliever who threw yesterday is marked tired and is not the starter', () => {
  const L = newLeague('tired');
  const me = L.clubs.RI!;
  const reliever = me.roster.find((p) => p.pitching && !me.rotation.includes(p.id))!;
  me.lastPitched = { [reliever.id]: { day: 4, pitches: 50 } };
  const team = toTeam(me, 'Reds', 5);
  const tired = team.players.filter((p) => p.tired);
  expect(tired.map((p) => p.id)).toEqual([reliever.id]);
  expect(team.startingPitcherId).not.toBe(reliever.id);
});

test('a new league is version 6', () => {
  expect(newLeague('v6').version).toBe(6);
});
```

- [ ] **Step 2: Run them to see them fail**

Run: `npm test -- tests/league/rest.test.ts`
Expected: FAIL (`lastStart` undefined, `toTeam` ignores the day, version 5).

- [ ] **Step 3: Implement**

`src/league/types.ts`: add to `Club`
```ts
  lastStart: Record<string, number>;                            // pitcher id -> day of his last start
  lastPitched: Record<string, { day: number; pitches: number }>; // any appearance, for tired relievers
```
and `version: 6`. `src/baseball/types.ts`: `Player` gains `pos?: Position; tired?: true` (the league's natural position rides into the engine in Task 2; `tired` is set per game day by `toTeam`).

`src/league/lineup.ts`:
```ts
export const REST_DAYS = 3;        // a starter goes on the fourth day: 1918 rotations were four men
export const TIRED_PITCHES = 35, TIRED_DAYS = 1; // a man who threw 35+ pitches yesterday sits today

// The engine's Team for today. The next rested man in rotation order starts; if nobody is rested, the
// most rested one does. Pitchers who worked yesterday are marked tired so the AI manager skips them.
export function toTeam(club: Club, name: string, day: number): Team {
  if (!club.rotation.length) throw new Error(`${club.id} has no rotation`);
  const order = club.rotation.map((_, i) => club.rotation[(club.nextStarter + i) % club.rotation.length]!);
  const rested = (id: string) => day - (club.lastStart[id] ?? -99) > REST_DAYS;
  const sp = order.find(rested) ?? order.reduce((a, b) => ((club.lastStart[a] ?? -99) <= (club.lastStart[b] ?? -99) ? a : b));
  const players = club.roster.map((p) => {
    const last = club.lastPitched[p.id];
    return p.pitching && p.id !== sp && last && day - last.day <= TIRED_DAYS && last.pitches >= TIRED_PITCHES ? { ...p, tired: true as const } : p;
  });
  const lineup = club.lineup.map((s) => (s.pos === 'P' ? { playerId: sp, pos: 'P' as Position } : s));
  return { id: club.id, name, players, lineup, startingPitcherId: sp };
}
```
`src/baseball/game.ts` `availablePitchers`: read the function first, then add `&& !p.tired` to its predicate.

`src/league/season.ts` `record()`: after the box is folded, for each side set `club.lastStart[s.teams[side].startingPitcherId] = L.day` and, for each pitching line of that side, `club.lastPitched[line.playerId] = { day: L.day, pitches: line.pitches }`. `nextStarter` keeps advancing as today. Every `toTeam(...)` call site passes `L.day` (season.ts has three; grep `toTeam(` across src and tests for others).

`src/league/league.ts`: every club literal gets `lastStart: {}`, `lastPitched: {}`; `offseason()` resets both to `{}` for every club. `src/save/save.ts`: `SAVE_VERSION = 6`, comment "Versions 1–5 are refused". `tests/league/helpers.ts`: `version: 6`. Update `tests/franchise/state.test.ts` and `tests/save.test.ts` version assertions (grep `version: 5`, `SAVE_VERSION` and `version 4`).

- [ ] **Step 4: Run the suite**

Run: `npm test` and `npm run build`
Expected: all green. The league calibration test (`tests/league/league-calibration.test.ts`) must still pass; if K or IP leaders shift, that is the intended effect and no tolerance changes.

- [ ] **Step 5: Commit**

```bash
git add src/league/types.ts src/baseball/types.ts src/baseball/game.ts src/league/lineup.ts src/league/season.ts src/league/league.ts src/save/save.ts tests/league/helpers.ts tests/league/rest.test.ts tests/franchise/state.test.ts tests/save.test.ts
git commit -m "Starters rest three days and tired relievers sit; save v6"
```

---

### Task 2: Position-specific defense and the out-of-position penalty

**Files:**
- Create: `src/baseball/defense.ts`
- Modify: `src/baseball/game.ts:65-77` (`avgRating` for fielding → `defenseRating`), `src/league/lineup.ts:13-30` (`autoLineup` fit)
- Test: `tests/baseball/defense.test.ts`

**Interfaces:**
- Produces: `POSITION_WEIGHT: Record<Exclude<Position, 'P'>, number>`, `outOfPosition(natural: Position | undefined, played: Position): number` (rating points lost, ≥ 0), `defenseRating(players: Player[], lineup: LineupSlot[]): number` (weighted effective fielding of the eight men in the field; the engine's `defenseFielding`).

- [ ] **Step 1: Write the failing tests**

```ts
// tests/baseball/defense.test.ts
import { expect, test } from 'vitest';
import { defenseRating, outOfPosition, POSITION_WEIGHT } from '../../src/baseball/defense';
import type { LineupSlot, Player, Position } from '../../src/baseball/types';

// Read src/baseball/fixtures.ts for the hitter factory and use it here; the shape below assumes hitter(id) returns a 65-everywhere Player.
import { hitter } from '../../src/baseball/fixtures';

const SPOTS = ['C', '1B', '2B', '3B', 'SS', 'LF', 'CF', 'RF', 'P'] as const;
const nine = (natural: Partial<Record<Position, Position>>): { players: Player[]; lineup: LineupSlot[] } => {
  const players = SPOTS.map((played, i) => ({ ...hitter(`h${i}`), pos: natural[played] ?? played }));
  return { players, lineup: SPOTS.map((played, i) => ({ playerId: `h${i}`, pos: played })) };
};

test('weights cover the eight fielders and sum to one', () => {
  expect(Object.values(POSITION_WEIGHT).reduce((a, b) => a + b, 0)).toBeCloseTo(1, 6);
  expect(POSITION_WEIGHT.SS).toBeGreaterThan(POSITION_WEIGHT['1B']);
});

test('a natural lineup of 65s rates 65; a left fielder at short costs the club defense', () => {
  const natural = nine({});
  expect(defenseRating(natural.players, natural.lineup)).toBeCloseTo(65, 6);
  const swapped = nine({ SS: 'LF' });
  expect(defenseRating(swapped.players, swapped.lineup)).toBeLessThan(63.5);
  expect(outOfPosition('LF', 'SS')).toBeGreaterThan(outOfPosition('SS', '2B'));
  expect(outOfPosition('2B', '2B')).toBe(0);
  expect(outOfPosition(undefined, 'C')).toBe(0); // engine-only fixtures have no natural position
});
```

- [ ] **Step 2: Run to see them fail** — `npm test -- tests/baseball/defense.test.ts` → FAIL (module missing).

- [ ] **Step 3: Implement**

```ts
// src/baseball/defense.ts
import type { LineupSlot, Player, Position } from './types';

// How much each glove matters to the fielding rating the matchup sees (sums to 1; the pitcher is not a fielder here).
export const POSITION_WEIGHT: Record<Exclude<Position, 'P'>, number> = { C: 0.10, '1B': 0.06, '2B': 0.14, '3B': 0.10, SS: 0.20, LF: 0.10, CF: 0.20, RF: 0.10 };

const IF = new Set<Position>(['1B', '2B', '3B', 'SS']), OF = new Set<Position>(['LF', 'CF', 'RF']);
// Rating points a man loses playing somewhere other than his position. ponytail: one table, no learning.
export function outOfPosition(natural: Position | undefined, played: Position): number {
  if (!natural || natural === played || played === 'P') return 0;
  if (played === 'C') return 25;
  if (played === '1B') return natural === 'C' ? 6 : 4;
  if (natural === 'C') return 8;
  if (IF.has(natural) && IF.has(played)) return natural === 'SS' ? 3 : played === 'SS' ? 10 : 6;
  if (OF.has(natural) && OF.has(played)) return played === 'CF' ? 8 : 2;
  return OF.has(played) ? 8 : 10; // infielder in the outfield / outfielder in the infield
}

export function defenseRating(players: Player[], lineup: LineupSlot[]): number {
  let total = 0;
  for (const slot of lineup) {
    if (slot.pos === 'P') continue;
    const p = players.find((x) => x.id === slot.playerId);
    if (!p) throw new Error(`no player ${slot.playerId}`);
    total += POSITION_WEIGHT[slot.pos] * (p.batting.fielding - outOfPosition(p.pos, slot.pos));
  }
  return total;
}
```
`src/baseball/game.ts` `targetsFor`: `defenseFielding: defenseRating(s.teams[fld].players, s.sides[fld].lineup)` (keep `avgRating` if `arm` still uses it; delete it if nothing does). `src/league/lineup.ts` `fit`: replace `- (p.pos === pos ? 0 : 8)` with the table: `batValue(p.batting) + (PREMIUM.has(pos) ? 0.3 : 0.1) * (p.batting.fielding - outOfPosition(p.pos, pos))`. Because `toTeam` passes roster objects, `pos` reaches the engine with no copying.

- [ ] **Step 4: Run the suite** — `npm test`, `npm run build`. The engine calibration tests (`tests/calibration.test.ts`, `tests/league/league-calibration.test.ts`) must stay inside their tolerances: generated clubs play natural positions, so the league rating should move less than a point. If a tolerance fails, report the numbers; do not widen the tolerance.

- [ ] **Step 5: Commit**
```bash
git add src/baseball/defense.ts src/baseball/game.ts src/league/lineup.ts tests/baseball/defense.test.ts
git commit -m "Position-weighted defense with an out-of-position table"
```

---

### Task 3: Holdouts

**Files:**
- Modify: `src/league/types.ts` (`LeaguePlayer.holdout?: { until: number }`), `src/franchise/economy.ts:51-65` (`renew`), `src/league/transactions.ts` (`Verdict`), `src/league/lineup.ts` (`available`; day-aware `autoLineup`, `autoRotation`, `rotationStale`, `lineupProblem`), `src/league/season.ts` (`ready()`), `src/league/league.ts` (`offseason`), `src/ui/OfficePage.tsx:50-53`, `src/ui/ClubPage.tsx`
- Test: `tests/franchise/holdout.test.ts`

**Interfaces:**
- Produces: `HOLDOUT_FLOOR = 0.6`, `HOLDOUT_DAYS_PER_POINT = 2`, `holdoutDays(expect, offer): number` in economy.ts; `renew` returns `{ ok: true; holdout?: number }` (days); `available(roster, day)` in lineup.ts. A holdout stays on the roster and under contract (`rosterProblem` is unchanged); he is simply unavailable until his day.

- [ ] **Step 1: Write the failing tests**

```ts
// tests/franchise/holdout.test.ts
import { expect, test } from 'vitest';
import { holdoutDays, renew, salaryFor } from '../../src/franchise/economy';
import { newLeague } from '../../src/league/league';
import { autoLineup, lineupProblem } from '../../src/league/lineup';
import { playDay } from '../../src/league/season';

test('an offer between 60% and 80% of expectation signs him but he holds out, longer the lower the offer', () => {
  const L = newLeague('holdout');
  const p = L.clubs.RI!.roster.find((x) => !x.pitching && !x.untouchable)!;
  const expect$ = salaryFor(p);
  expect(renew(L, p.id, Math.round(expect$ * 0.5)).ok).toBe(false);
  const r = renew(L, p.id, Math.round(expect$ * 0.7));
  expect(r.ok && r.holdout).toBe(holdoutDays(expect$, Math.round(expect$ * 0.7)));
  expect(holdoutDays(expect$, Math.round(expect$ * 0.7))).toBeGreaterThan(holdoutDays(expect$, Math.round(expect$ * 0.78)));
  expect(holdoutDays(expect$, expect$)).toBe(0);
  expect(p.holdout).toEqual({ until: holdoutDays(expect$, Math.round(expect$ * 0.7)) });
  renew(L, p.id, expect$);
  expect(p.holdout).toBeUndefined();
});

test('a holdout is out of the lineup until his day, then eligible again', () => {
  const L = newLeague('holdout-lineup');
  const me = L.clubs.RI!;
  const p = me.roster.find((x) => !x.pitching && !x.untouchable && me.lineup.some((s) => s.playerId === x.id))!;
  renew(L, p.id, Math.round(salaryFor(p) * 0.65));
  const until = p.holdout!.until;
  expect(until).toBeGreaterThan(0);
  expect(lineupProblem(me, 0)).toMatch(/holding out/);
  autoLineup(me, 0);
  expect(me.lineup.some((s) => s.playerId === p.id)).toBe(false);
  L.phase = 'season';
  while (L.day <= until) playDay(L, true);
  expect(p.holdout).toBeUndefined();
  expect(lineupProblem(me, L.day)).toBeNull();
});
```

- [ ] **Step 2: Run to see them fail** — `npm test -- tests/franchise/holdout.test.ts` → FAIL.

- [ ] **Step 3: Implement**

`src/franchise/economy.ts`:
```ts
export const HOLDOUT_FLOOR = 0.6;          // below this share of his expectation he will not sign at all
export const HOLDOUT_DAYS_PER_POINT = 2;   // each percentage point under the 20% cut costs two days of the season
export const holdoutDays = (expect: number, offer: number) =>
  Math.max(0, Math.round(((1 - MAX_CUT) - offer / expect) * 100 * HOLDOUT_DAYS_PER_POINT));
```
In `renew`: replace the `MAX_CUT` refusal with `if (offer < expect * HOLDOUT_FLOOR) return { ok: false, reason: 'He will not sign for that.' };`; after recording the salary: `const days = holdoutDays(expect, offer); if (days) p.holdout = { until: days }; else delete p.holdout;` and `return { ok: true, holdout: days || undefined };`. Extend `Verdict` in transactions.ts to `{ ok: true; holdout?: number } | { ok: false; reason: string }`. The local pay-cut culture rule stays.

`src/league/lineup.ts`: `export const available = (roster: LeaguePlayer[], day: number) => roster.filter((p) => !p.holdout || p.holdout.until <= day);` `autoLineup(club, day = 0)` and `autoRotation(club, day = 0)` pick from `available(club.roster, day)` (hitters and pitchers of it); `lineupProblem(club, day = 0)` returns `` `${p.name} is holding out` `` for a slot whose man is holding out on `day`; `rotationStale(club, day = 0)` is also true when a rotation man is holding out. `src/league/season.ts` `ready(club)`: first delete `p.holdout` for every roster man with `until <= L.day`, then pass `L.day` to all four lineup functions (AI clubs never hold out, so this only ever touches the Reds). `offseason()` deletes every `holdout`. Other call sites of `autoLineup`/`autoRotation` keep the default `day = 0` (in winter a holdout is excluded, which is right).

UI: `OfficePage.tsx` beside the Sign button, when the typed offer is under `salaryFor(p) * (1 - MAX_CUT)`, show `` `He will hold out ${holdoutDays(expect, offer)} days.` `` (plain text; this is a contract fact, not a hidden value). `ClubPage.tsx` hitters and pitchers tables: append " (holding out)" to a name while `p.holdout`.

- [ ] **Step 4: Run the suite** — `npm test`, `npm run build`. `tests/franchise/office.test.ts` asserts the old 20% refusal; change that assertion to the holdout band and say so in the report.
- [ ] **Step 5: Commit**
```bash
git add src/league/types.ts src/franchise/economy.ts src/league/transactions.ts src/league/lineup.ts src/league/season.ts src/league/league.ts src/ui/OfficePage.tsx src/ui/ClubPage.tsx tests/franchise/holdout.test.ts tests/franchise/office.test.ts
git commit -m "Holdouts: a low offer signs him for a share of the season"
```

---

### Task 4: AI clubs keep their own books and buy in winter

**Files:**
- Modify: `src/franchise/economy.ts:118-150` (`settle`), `src/league/league.ts:71-96` (`offseason`), `src/league/transactions.ts` (export `needs`)
- Test: `tests/franchise/ai-books.test.ts`

**Interfaces:**
- Produces: `aiNet(L, clubId): number` in economy.ts (replaces `AI_NET`, which is deleted); `aiWinter(L): void` in league.ts, called by `offseason()` after the `fillRoster` loop and before `L.pool = genPool(L)`; `AI_BUY_RESERVE = 3000`, `AI_MAX_BUYS = 2` in economy.ts.

- [ ] **Step 1: Write the failing tests**

```ts
// tests/franchise/ai-books.test.ts
import { expect, test } from 'vitest';
import { aiNet } from '../../src/franchise/economy';
import { city } from '../../src/league/cities';
import { newLeague, offseason } from '../../src/league/league';
import { simSeason } from '../../src/league/season';

test('an AI club nets its own gate less payroll, in a plausible band, and richer parks net more', () => {
  const L = newLeague('ai-books');
  L.phase = 'season';
  simSeason(L);
  const nets = Object.fromEntries(Object.keys(L.clubs).filter((id) => id !== 'RI').map((id) => [id, aiNet(L, id)]));
  for (const [id, net] of Object.entries(nets)) {
    expect(net, id).toBeGreaterThan(-15000);
    expect(net, id).toBeLessThan(40000);
  }
  const byTier = (t: number) => Object.entries(nets).filter(([id]) => city(id).tier === t).map(([, n]) => n);
  expect(Math.max(...byTier(3))).toBeGreaterThan(Math.min(...byTier(1)));
});

test('a rich AI club buys from the pool in winter when it has a need and cash; a broke one sells', () => {
  const L = newLeague('ai-winter');
  L.phase = 'season';
  simSeason(L);
  const bos = L.clubs.BOS!, bal = L.clubs.BAL!;
  bos.cash = 50000; bal.cash = -2000;
  const bosBefore = bos.roster.map((p) => p.id), balBefore = bal.roster.length;
  offseason(L);
  expect(bos.roster.some((p) => !bosBefore.includes(p.id) && !p.local)).toBe(true);
  expect(bos.cash).toBeLessThan(50000);
  expect(bal.cash).toBeGreaterThanOrEqual(0);
  expect(bal.roster.length).toBeLessThanOrEqual(balBefore);
});
```
(`offseason` runs `fillRoster` after `retire`, which can add free amateurs to Boston too; the purchase assertion checks for a man who is not `local` and was not on the roster, which only `aiWinter` produces. If `fillRoster`'s generated men also lack `local`, mark `aiWinter` purchases instead with the `Deal` ledger or check `L.pool` ids before/after; report which.)

- [ ] **Step 2: Run to see them fail** — `npm test -- tests/franchise/ai-books.test.ts` → FAIL (`aiNet` missing).

- [ ] **Step 3: Implement**

`src/franchise/economy.ts`:
```ts
export const AI_BUY_RESERVE = 3000, AI_MAX_BUYS = 2;
// An AI club's season on the Reds' terms: its gate at 35 cents less the visitor share, road shares, concessions,
// advertising and dues, travel, and full salaries. ponytail: no AI loans; a losing year is paid for with a sale.
export function aiNet(L: LeagueState, clubId: string): number {
  const club = L.clubs[clubId]!;
  let net = ADVERTISING - DUES;
  for (const r of L.results) {
    const gate = Math.round(r.attendance * 35 / 100);
    if (r.home === clubId) net += gate - Math.round(gate * VISITOR_SHARE) + Math.round(r.attendance * CONCESSION_PER_HEAD / 100);
    else if (r.away === clubId) net += Math.round(gate * VISITOR_SHARE) - TRAVEL_PER_ROAD_GAME;
  }
  return net - club.roster.reduce((sum, p) => sum + salaryFor(p), 0);
}
```
In `settle`: replace the `AI_NET` line with `for (const club of Object.values(L.clubs)) if (club.id !== L.userClub) club.cash += aiNet(L, club.id);`. Delete `AI_NET`.

`src/league/league.ts`:
```ts
// Winter dealing for the seven rivals: a club in the red sells its most valuable man outside the lineup and
// rotation to the pool at the bid; a club with cash buys the best pool man who beats its weakest starter at a
// position, up to AI_MAX_BUYS, keeping AI_BUY_RESERVE. Richest club first. No RNG.
export function aiWinter(L: LeagueState): void {
  const rivals = Object.values(L.clubs).filter((c) => c.id !== L.userClub).sort((a, b) => b.cash - a.cash);
  for (const club of rivals) {
    while (club.cash < 0) {
      const spare = club.roster.filter((p) => !club.rotation.includes(p.id) && !club.lineup.some((s) => s.playerId === p.id)).sort((a, b) => value(b) - value(a))[0];
      if (!spare) break;
      club.roster = club.roster.filter((p) => p !== spare);
      club.cash += bidPrice(value(spare));
      L.pool.push({ ...spare, ask: askPrice(value(spare)) });
    }
    let buys = 0;
    for (const p of [...L.pool].sort((a, b) => value(b) - value(a))) {
      if (buys >= AI_MAX_BUYS || club.roster.length >= ROSTER_MAX) break;
      if (club.cash - p.ask < AI_BUY_RESERVE || !needs(club, p)) continue;
      L.pool = L.pool.filter((x) => x !== p);
      const { ask: _ask, ...signed } = p;
      club.roster.push(signed);
      club.cash -= p.ask;
      buys++;
    }
  }
}
```
Call `aiWinter(L)` in `offseason()` after the `fillRoster` loop and before `L.pool = genPool(L)`, so purchases come from the pool the Reds also saw all year; men sold to the pool vanish with the regeneration (the abstract minors swallow them). Export `needs` from transactions.ts.

- [ ] **Step 4: Run the suite** — `npm test`, `npm run build`. `tests/franchise/economy.test.ts` may reference `AI_NET`; update it to `aiNet`.
- [ ] **Step 5: Commit**
```bash
git add src/franchise/economy.ts src/league/league.ts src/league/transactions.ts tests/franchise/ai-books.test.ts tests/franchise/economy.test.ts
git commit -m "AI clubs keep their own books and deal with the pool in winter"
```

---

### Task 5: Sunday law by city

**Files:**
- Modify: `src/league/types.ts` (`City.sundayFrom: number | null`), `src/league/cities.ts`, `src/league/schedule.ts:28-48` (balancing), `src/league/league.ts` (both `makeSchedule` calls), `src/franchise/economy.ts:84-96` (`attendanceFor`)
- Test: `tests/league/sunday-law.test.ts`

**Interfaces:**
- Produces: `City.sundayFrom` (first year paid Sunday home ball is legal; `null` = never in the slice); `sundayOpen(id, year, flags): boolean` in cities.ts; `makeSchedule(ids, year, closed: string[] = [])`.

- [ ] **Step 1: Write the failing tests**

```ts
// tests/league/sunday-law.test.ts
import { expect, test } from 'vitest';
import { attendanceFor } from '../../src/franchise/economy';
import { CITIES, sundayOpen } from '../../src/league/cities';
import { newLeague } from '../../src/league/league';
import { isSunday, makeSchedule } from '../../src/league/schedule';

test('closed cities host no Sunday games and open ones stay within two of each other', () => {
  const ids = CITIES.map((c) => c.id);
  const closed = ids.filter((id) => !sundayOpen(id, 1920, {}));
  expect(closed).toEqual(expect.arrayContaining(['PIT', 'BOS', 'RI']));
  const games = makeSchedule(ids, 1920, closed);
  const sundays = games.filter((g) => isSunday(g.date));
  for (const id of closed) {
    for (const g of sundays.filter((g) => g.home === id)) expect(closed, `${id} hosts ${g.away} on a Sunday`).toContain(g.away);
  }
  const counts = ids.filter((id) => !closed.includes(id)).map((id) => sundays.filter((g) => g.home === id).length);
  expect(Math.max(...counts) - Math.min(...counts)).toBeLessThanOrEqual(2);
  expect(games).toHaveLength(154 * 4);
  for (const id of ids) {
    expect(games.filter((g) => g.home === id)).toHaveLength(77);
    for (const other of ids) if (other !== id) expect(games.filter((g) => g.home === id && g.away === other)).toHaveLength(11);
  }
});

test('Boston opens in 1929, Buffalo in 1919, Pittsburgh never; Rhode Island by the flag', () => {
  expect(sundayOpen('BOS', 1928, {})).toBe(false);
  expect(sundayOpen('BOS', 1929, {})).toBe(true);
  expect(sundayOpen('BUF', 1918, {})).toBe(false);
  expect(sundayOpen('BUF', 1919, {})).toBe(true);
  expect(sundayOpen('PIT', 1930, {})).toBe(false);
  expect(sundayOpen('RI', 1930, {})).toBe(false);
  expect(sundayOpen('RI', 1918, { sunday_ball: true })).toBe(true);
});

test('an open park draws the Sunday crowd; a closed one draws a weekday crowd', () => {
  const L = newLeague('sunday-law');
  const sundayDate = L.schedule.find((g) => isSunday(g.date))!.date;
  const weekdayDate = L.schedule.find((g) => !isSunday(g.date))!.date;
  expect(attendanceFor(L, 'CLE', sundayDate)).toBeGreaterThan(attendanceFor(L, 'CLE', weekdayDate));
  expect(attendanceFor(L, 'PIT', sundayDate)).toBe(attendanceFor(L, 'PIT', weekdayDate));
});
```

- [ ] **Step 2: Run to see them fail** — `npm test -- tests/league/sunday-law.test.ts` → FAIL.

- [ ] **Step 3: Implement**

`src/league/cities.ts`: add `sundayFrom` to each city: RI `null` (the flag), STL 1902, BOS 1929, BUF 1919, PIT `null`, CLE 1918, CIN 1902, BAL 1919 (research above; keep the placeholder comments). Add
```ts
// Paid Sunday home ball. Rhode Island is the Rocky Point story (M3): open only once the hearing is won.
export const sundayOpen = (id: string, year: number, flags: Record<string, true>): boolean =>
  id === 'RI' ? !!flags.sunday_ball : city(id).sundayFrom !== null && year >= city(id).sundayFrom!;
```
`src/league/schedule.ts` `makeSchedule(ids, year, closed = [])`: before the balancing loop, for every Sunday game hosted by a closed club whose visitor is open, swap home/away with it and with the matching weekday return date (the same two-line swap the balancer uses); then balance only among open clubs (`counts` over `ids.filter(open)`; never pick a `sunday` whose `away` is closed). A Sunday game between two closed clubs stays where it is. The 77/77 and 11-per-pair invariants hold because every swap is paired with its return date.

`src/league/league.ts`: both `makeSchedule` calls pass `CITIES.filter((c) => !sundayOpen(c.id, year, L.story.flags)).map((c) => c.id)` as `closed`. Rhode Island is "closed" for scheduling while the flag is unset. **This changes M3's Rocky Point economics:** the Reds' Sunday home dates move to open rivals' parks (a road share instead of the resort's 60%). Keep `ROCKY_POINT` in `attendanceFor`/`gameMoney` for any Sunday home date that remains (a Sunday against another closed club), and run `tests/franchise/economy.test.ts`; if it pinned Reds Sunday home counts or Rocky Point totals, update the expectation and say so in the report.

`src/franchise/economy.ts` `attendanceFor`: the AI Sunday factor becomes `sundayOpen(home, L.year, L.story.flags) ? AI_SUNDAY : 1` (the Reds' branch is unchanged).

- [ ] **Step 4: Run the suite** — `npm test`, `npm run build`.
- [ ] **Step 5: Commit**
```bash
git add src/league/types.ts src/league/cities.ts src/league/schedule.ts src/league/league.ts src/franchise/economy.ts tests/league/sunday-law.test.ts tests/league/schedule.test.ts tests/franchise/economy.test.ts
git commit -m "Sunday law by city: closed parks host no Sunday ball"
```

---

### Task 6: The pennant is an event (clinch stop, pennant flags, `won_pennant` readouts)

**Files:**
- Create: `content/notices.yaml` (Codex creates it with the placeholder entry below; Fable writes the prose in Task 7)
- Modify: `src/franchise/loop.ts:30-66` (`clinched`, `stopAtEvent`, `toFall`, `toWinter`), `src/story/types.ts` (`StoryState.notice`, `Readout.when.won_pennant`, `Notices`), `src/story/content.ts` + `src/story/check.ts` (load and check notices), `src/story/readouts.ts`, `src/league/league.ts` + `tests/league/helpers.ts` (`notice: null`), `src/ui/App.tsx`
- Test: `tests/franchise/clinch.test.ts`

**Interfaces:**
- Produces: `clinched(L): boolean`; `StoryState.notice: 'clinch' | null`; `NOTICES: Notices` with `Notices = Record<'clinch', { title: string; text: string }>`; flags `clinched_${year}` (set once; the sim stops that day), `pennant_${year}` (set in `toFall` when the Reds finish first), `pennant_last` (set in `toWinter` when last season's pennant was the Reds', else deleted); readout condition `won_pennant: true` ⇔ `flags[`pennant_${L.year}`]`.

- [ ] **Step 1: Write the failing tests**

```ts
// tests/franchise/clinch.test.ts
import { expect, test } from 'vitest';
import { advance, clinched, toFall } from '../../src/franchise/loop';
import { newLeague } from '../../src/league/league';
import { standings } from '../../src/league/season';
import type { LeagueState } from '../../src/league/types';

// Nothing is rigged: the rivals are given poor players and the engine plays every game out.
function weaken(L: LeagueState, ids: string[], to: number) {
  for (const id of ids) for (const p of L.clubs[id]!.roster) {
    for (const k of Object.keys(p.batting) as (keyof typeof p.batting)[]) p.batting[k] = to;
    if (p.pitching) for (const k of Object.keys(p.pitching) as (keyof typeof p.pitching)[]) p.pitching[k] = to;
  }
}
function run(L: LeagueState) {
  L.phase = 'season';
  while (L.phase === 'season' && !L.story.notice) {
    advance(L);
    L.story.pending = null; L.story.storyGame = null;
    if (L.live) { L.live = null; L.story.moment = null; L.day++; } // a Moment cannot fire for a runaway winner; this guards the loser test
  }
}

test('the sim stops once when the Reds clinch, then runs on; the fall sets the pennant flag', () => {
  const L = newLeague('clinch');
  weaken(L, Object.keys(L.clubs).filter((id) => id !== 'RI'), 40);
  run(L);
  expect(L.story.notice).toBe('clinch');
  expect(clinched(L)).toBe(true);
  expect(L.day).toBeLessThan(154);
  expect(L.story.flags.clinched_1918).toBe(true);
  L.story.notice = null;
  run(L);
  expect(L.story.notice).toBeNull();
  expect(standings(L)[0]!.id).toBe('RI');
  expect(L.story.flags.pennant_1918).toBe(true);
});

test('a club that never clinches never sees the notice', () => {
  const L = newLeague('no-clinch');
  weaken(L, ['RI'], 30);
  run(L);
  expect(L.story.notice).toBeNull();
  expect(L.story.flags.clinched_1918).toBeUndefined();
  expect(L.story.flags.pennant_1918).toBeUndefined();
});
```
(If skipping a live game by `L.day++` leaves the day's other games unplayed and breaks `toFall`, use `simStoryGame`/`closeLive` from loop.ts and season.ts instead; the point is only that the loser test must not hang on a Moment. Report what was needed.)

- [ ] **Step 2: Run to see them fail** — FAIL (`clinched` missing).

- [ ] **Step 3: Implement**

`src/franchise/loop.ts`:
```ts
// Clinched when no rival can reach the Reds' win total with every game he has left.
export function clinched(L: LeagueState): boolean {
  const table = standings(L), me = table.find((r) => r.id === L.userClub)!;
  return table.filter((r) => r.id !== L.userClub).every((r) => {
    const left = L.schedule.filter((g) => g.day >= L.day && (g.home === r.id || g.away === r.id)).length;
    return r.w + left < me.w;
  });
}
```
In `stopAtEvent`, after the `L.day >= DAYS` check and before the story-game offer:
```ts
  if (L.story.notice) return true;
  if (!L.story.flags[`clinched_${L.year}`] && clinched(L)) {
    L.story.flags[`clinched_${L.year}`] = true;
    L.story.notice = 'clinch';
    return true;
  }
```
In `toFall`, before `settle`: `if (standings(L)[0]!.id === L.userClub) L.story.flags[`pennant_${L.year}`] = true;`. In `toWinter`, after `offseason(L)`: `if (L.history.at(-1)?.pennant === L.userClub) L.story.flags.pennant_last = true; else delete L.story.flags.pennant_last;`.

`src/story/types.ts`: `StoryState.notice: 'clinch' | null`; `Readout.when` gains `won_pennant?: true`; `export type Notices = Record<'clinch', { title: string; text: string }>`. `src/story/readouts.ts` `select`: add `|| (w.won_pennant && !flags[`pennant_${L.year}`])` to the skip condition. `src/story/content.ts`: `export const NOTICES: Notices = parse(notices)` from `content/notices.yaml` (same import style as `story-games.yaml`); `check.ts` requires `clinch.title` and `clinch.text` to be non-empty strings. `content/notices.yaml` placeholder: `clinch: { title: The pennant, text: PLACEHOLDER (Task 7) }`. Initialise `notice: null` in `newLeague` and `tests/league/helpers.ts`.

`src/ui/App.tsx`: when `L.story.notice`, render a page with `NOTICES[notice].title` as the heading, the text as `.prose`, and one button "Go on" that sets `notice = null` through `run`. Follow the pattern the story-game page uses for its stop.

- [ ] **Step 4: Run the suite** — `npm test`, `npm run build`.
- [ ] **Step 5: Commit**
```bash
git add src/franchise/loop.ts src/story/types.ts src/story/content.ts src/story/check.ts src/story/readouts.ts src/ui/App.tsx content/notices.yaml src/league/league.ts tests/league/helpers.ts tests/franchise/clinch.test.ts
git commit -m "The pennant is an event: clinch stop, pennant flags, won_pennant readouts"
```

---

### Task 7: Content for the three seasons (Fable)

**Files:**
- Create: `content/cards/the-lively-ball.md` (`fixed: true`, `years: [1920, 1920]`, `phase: winter`, `priority: 100`, `repeat: never`; three options, one free: the new ball and the end of the spitball told through Draper's billboard in right, the price of baseballs and what the pitchers say; the ambition option is a billboard deal with `earn` and a flag; the roots option keeps the right-field boards for the kids at a `spend`; the money option raises nothing but marks the club waiting to see), `content/cards/pennant-winter.md` (`requires: { flag: [pennant_last] }`, `phase: winter`, `priority: 95`, `repeat: yearly`, `years: [1919, 1930]`: what the neighborhood, Ashworth and the men each want from a pennant winter; the money option is a raise in the price with `set_flag: pennant_price` and `adjust: loyalty −2`; the roots option a free day with `spend` and `culture +1`; the ambition option Ashworth's dinner with `earn` and `ashworth_pennant`), `content/cards/the-strike.md` (`phase: block`, `years: [1919, 1930]`, `repeat: never`, `priority: 75`: the mills stop; the club chooses who gets in and who gets paid; the `exploit` option exists and costs)
- Modify: `content/notices.yaml` (`clinch` title and text), `content/readouts.yaml` (`won_pennant` entries: gate, bleachers, clubhouse, and gus with `trust_above: 60`; one `lost_pennant_race` bleachers entry), `content/letters.yaml` (1920 prose gains the lively-ball line in all three tones)

- [ ] **Step 1: Write the cards, notice and readouts** in the voice of the existing content (no year in prose; every card touches a roster, an account or a person).
- [ ] **Step 2: Run** `npm test -- tests/story` → the content checker passes. Add one assertion to `tests/story/scheduler.test.ts`: with `L.year = 1920` and nothing dealt, `deal(L, 'winter')` returns `the-lively-ball`.
- [ ] **Step 3: Commit**
```bash
git add content/cards/the-lively-ball.md content/cards/pennant-winter.md content/cards/the-strike.md content/notices.yaml content/readouts.yaml content/letters.yaml tests/story/scheduler.test.ts
git commit -m "Content: the lively ball, a pennant winter, the strike, clinch notice, pennant readouts"
```

---

### Task 8: Strategy bots over 1918–1930 and the numbers gate

**Files:**
- Create: `tests/bots/run.ts`, `tests/bots/policies.ts`, `tests/bots/bots.test.ts`; the test writes `docs/m4-build/bots-report.md`; `package.json` gains `"bots": "vitest run tests/bots"`.
- Reference: the M3 probe at `D:/UserData/Alex/Temp/claude/C--Users-Alex/33b6c540-45a2-4b1e-91b0-ecc47128c111/scratchpad/probe/m3probe.test.ts` shows how to drive the phase machine headless, clear story games, and play a Moment with a policy. Copy what is needed into `run.ts`; do not import from the scratchpad.

**Interfaces:**
- Produces: `runBot(name: BotName, seed: string): BotResult` with `BotResult = { name: BotName; seed: string; years: { year: number; w: number; l: number; pennant: string; cash: number; band: 0 | 1 | 2 | 3; foreclosed: boolean }[]; leagueRpg: Record<number, number> }`; `POLICIES: Record<BotName, Policy>` with `BotName = 'roots' | 'ambition' | 'money' | 'flipflop' | 'gouger' | 'seller' | 'borrower'` and `Policy = { price: number; winter(L: LeagueState): void; option(L: LeagueState, card: Card): Option; moment(balls: number, strikes: number): 'take' | 'swing' }`.

- [ ] **Step 1: Write the policies and runner**

```ts
// tests/bots/policies.ts (shape; the helpers are described below)
export const POLICIES: Record<BotName, Policy> = {
  roots:    path('roots', 25),
  ambition: path('ambition', 35),
  money:    path('money', 50),
  flipflop: { ...path('roots', 35), option: (L, card) => byPath(L, card, L.story.choices.length % 2 ? 'roots' : 'money') },
  gouger:   path('money', 75),
  seller:   { ...path('money', 35), winter: (L) => { defaultWinter(L, 35); sellBest(L); } },
  borrower: { ...path('roots', 35), winter: (L) => { defaultWinter(L, 35); if (L.year >= 1929) borrow(L, LOAN_CAP - outstanding(L)); } },
};
```
`path(p, price)`: `option` = `byPath(L, card, p)` = the first option whose `path === p` that `optionProblem(L, card, o)` allows, else the first allowed option; `winter` = `defaultWinter(L, price)`: `renew` every man at `salaryFor` (no holdouts), `fundRepairs(L, 1)` when cash > 2000 and `L.franchise.hidden.infrastructure < 60` (bots may read hidden values; screens may not), `setPrice(L, price)`. `sellBest`: sell the highest-`value` non-untouchable hitter to the richest rival with `sellPlayer(L, id, pid, false, true)` (read transactions.ts for the exact signature). `moment`: 'take' on 3-0 and 2-0, otherwise 'swing' (the same read for every bot; Moments are not what the bots test). The runner drives: `toOpening`; while `phase !== 'over'`: if `pending` → `choose(L, policy.option(L, card).id)`; else if `storyGame` → `simStoryGame`; else if `moment` or `live` → play pitches with `policy.moment` until the Moment resolves and the game closes (copy the probe's loop); else if `notice` → `notice = null`; else by phase: `winter` → `policy.winter(L)`, `toOpening`; `opening` → `playBall`; `season` → `advance`; `fall` → `toWalk`; `walk` → `toWinter`. After each `toWalk`, record the year from `L.history.at(-1)` (or `standings` if the history entry lands in `toWinter`; read loop.ts), `L.clubs.RI.cash`, `shownBand(L.franchise.hidden)`, `foreclosed` from `L.story.flags.foreclosed`, and the league runs per game from `L.results` for that year.

- [ ] **Step 2: Write the gate test**

```ts
// tests/bots/bots.test.ts
import { mkdirSync, writeFileSync } from 'node:fs';
import { describe, expect, test } from 'vitest';
import { getEra } from '../../src/baseball/eras';
import { POLICIES, type BotName } from './policies';
import { runBot } from './run';

const SEEDS = ['m4-a', 'm4-b'];
const results = (Object.keys(POLICIES) as BotName[]).flatMap((name) => SEEDS.map((seed) => runBot(name, seed)));
const pennants = (r: (typeof results)[number]) => r.years.reduce<Record<string, number>>((m, y) => ((m[y.pennant] = (m[y.pennant] ?? 0) + 1), m), {});

describe('strategy bots 1918-1930', { timeout: 900_000 }, () => {
  test('report', () => {
    mkdirSync('docs/m4-build', { recursive: true });
    const lines = ['# Strategy bots, 1918-1930', '', '| bot | seed | Reds pennants | 1930 cash | 1930 band | foreclosed | top rival |', '|---|---|---|---|---|---|---|'];
    for (const r of results) {
      const counts = pennants(r);
      const rival = Object.entries(counts).filter(([id]) => id !== 'RI').sort((a, b) => b[1] - a[1])[0];
      const last = r.years.at(-1)!;
      lines.push(`| ${r.name} | ${r.seed} | ${counts.RI ?? 0} | ${last.cash} | ${last.band} | ${r.years.find((y) => y.foreclosed)?.year ?? '-'} | ${rival ? rival.join(' x') : '-'} |`);
    }
    lines.push('', '## League runs per game vs era', '', '| year | sim | era |', '|---|---|---|');
    for (let y = 1918; y <= 1930; y++) {
      const sims = results.map((r) => r.leagueRpg[y]).filter((x): x is number => x !== undefined);
      if (sims.length) lines.push(`| ${y} | ${(sims.reduce((a, b) => a + b, 0) / sims.length).toFixed(2)} | ${getEra(y).runsPerGame.toFixed(2)} |`);
    }
    writeFileSync('docs/m4-build/bots-report.md', lines.join('\n') + '\n');
    expect(results).toHaveLength(14);
  });

  test('no careful strategy forecloses before 1929; the seller and the borrower may', () => {
    for (const r of results.filter((r) => !['seller', 'borrower'].includes(r.name))) {
      expect(r.years.find((y) => y.foreclosed)?.year ?? 1931, `${r.name}/${r.seed}`).toBeGreaterThanOrEqual(1929);
    }
  });

  test('roots and money read as different towns by 1930', () => {
    for (const seed of SEEDS) {
      const roots = results.find((r) => r.name === 'roots' && r.seed === seed)!, money = results.find((r) => r.name === 'money' && r.seed === seed)!;
      expect(roots.years.at(-1)!.band - money.years.at(-1)!.band, seed).toBeGreaterThanOrEqual(1);
    }
  });

  test('no club owns the league: no rival wins more than 7 of 13, the Reds at most 5', () => {
    for (const r of results) for (const [id, n] of Object.entries(pennants(r))) expect(n, `${r.name}/${r.seed}/${id}`).toBeLessThanOrEqual(id === 'RI' ? 5 : 7);
  });

  test('league scoring tracks the era within 10% every year', () => {
    for (let y = 1918; y <= 1930; y++) {
      const sims = results.map((r) => r.leagueRpg[y]).filter((x): x is number => x !== undefined);
      if (!sims.length) continue;
      const mean = sims.reduce((a, b) => a + b, 0) / sims.length;
      expect(Math.abs(mean / getEra(y).runsPerGame - 1), `${y}`).toBeLessThan(0.10);
    }
  });

  test('1929-30: the money bot ends solvent', () => {
    for (const r of results.filter((r) => r.name === 'money')) expect(r.years.at(-1)!.cash, r.seed).toBeGreaterThanOrEqual(0);
  });
});
```
Fourteen headless runs of thirteen seasons is about 14 × 13 × 0.8 s ≈ 2.5 minutes. The `describe` timeout covers it; `npm test` still runs the file.

- [ ] **Step 3: Run** `npm run bots`. Expected on the first run: the report is written; some gate tests FAIL. That is the point: append a "## Failures" section to `docs/m4-build/bots-report.md` listing each failing assertion with its numbers, and STOP. Do not tune constants in this task.
- [ ] **Step 4: Commit the bots, the report and the gate as it stands**
```bash
git add tests/bots/run.ts tests/bots/policies.ts tests/bots/bots.test.ts docs/m4-build/bots-report.md package.json
git commit -m "Strategy bots over 1918-1930 with the numbers gate (report attached)"
```
(If the gate fails, `npm test` is red at this commit. Say so in the commit body; Task 9 makes it green.)

---

### Task 9: Tune to the bots (Fable decides, Codex turns the knobs)

**Files:** `src/franchise/economy.ts` constants, `src/league/generate.ts` (`genRoster` means, `genPool` mean 52), `src/league/league.ts` (`fillRoster` means 50/60 and 55/58), `src/league/development.ts`, `src/franchise/hidden.ts` (`yearly`, `winter`), `docs/m4-build/bots-report.md`, this plan's Execution notes.

- [ ] **Step 1: Read `docs/m4-build/bots-report.md` and map each failing gate to its knob:**
  - A careful bot forecloses early → `BASE_DRAW`, `DEMAND_GROWTH`, `SALARY_BASE`, `DUES`/`ADVERTISING` (money is tight by design; move salaries before the gate).
  - Roots and money read the same → the yearly loyalty/culture moves in `hidden.ts` (`yearly`, `winter`) and the price bands `AFFORDABLE`/`EXCLUSIONARY`; never the band edges or the Standard formula (spec §5 is law).
  - One rival owns the league → `genRoster` tier means (65 + 3·(tier − 2)), `AI_BUY_RESERVE`, `genPool` mean; the Reds too strong → their `fillRoster` means.
  - Runs per game drift from the era → `fillRoster`/`genPool` means (replacement drift) and `development.ts` growth/decline; never the era table.
  - The money bot ends 1930 in the red → the 1929–30 `0.85` demand factor and `COALITION_LAST_YEAR` stay (spec §8); move `BASE_DRAW` or `SALARY_BASE`.
- [ ] **Step 2: Change at most three constants per iteration, rerun `npm run bots`, append the iteration (what moved, old → new, which gates flipped) to the report. At most three iterations; if the gate is still red, stop and report: that is a design finding for Alex, not a knob.**
- [ ] **Step 3: `npm test` (the whole suite, including the M1–M3 calibration and economy tests) and `npm run build` green.**
- [ ] **Step 4: Commit each iteration separately** with explicit paths and the gate state in the message.

---

### Task 10: Review, docs, handoff

**Files:** game repo `README.md` (M4 section: what rests, what defense does, holdouts, AI books, the Sunday law table, the clinch, the bots and `npm run bots`; carry-overs to M5), `docs/m4-build/` (ledger `progress.md`, task reports, `final-review.md`), `.git/sdd/progress.md`; docs repo `HANDOFF.md` and this plan's Execution notes.

- [ ] **Step 1:** Codex adversarial review of the whole branch (`adversarial-review --wait --base bb92ddd --scope branch`), focus: determinism (no new `rng.league` draws in-season), hidden values on screen, the AI books' arithmetic, holdout edge cases (a holdout traded, sold or released mid-holdout; a holdout in the rotation; a holdout and `rosterProblem`), schedule integrity after the Sunday swaps (77 home / 77 away, 11 per pair each way), the clinch when the Reds are tied. Fix every finding in one wave.
- [ ] **Step 2:** README and ledger; HANDOFF "Latest" section; Execution notes in this plan.
- [ ] **Step 3:** Commit and push both repos.

## Acceptance (the gate, spec §13)

- Codex-verifiable: `npm test` green including `tests/bots`; `docs/m4-build/bots-report.md` shows every gate passing.
- Alex: play 1918–1920 in the browser (a new league; v5 saves are refused). Check: a starter never starts on short rest (box scores); a man playing out of position hurts the defense (move your left fielder to short on the Club page and watch the errors); offer a man 70% and watch him sit; the clinch stop if you win; the lively-ball card in the 1920 winter; Sunday dates at Boston and Pittsburgh are gone from the schedule; AI clubs' cash moves on the League page. Then the two human questions: is it fun, and can you say why the park changed. Name the six clubs if you have not.

## Execution notes

(Filled in during the build: every plan correction, in the order found.)

1. **Task 2:** the plan's fit expression scaled the out-of-position penalty by the glove weight (0.1/0.3), so an outfielder at first base cost 0.4 fit points and 46 of 64 fielders played out of position; calibration broke. Fix: the penalty is subtracted unscaled (fit points, like the old flat 8). Two seed-coupled Moment tests reseeded (`moment-live-2` → `moment-live-6`).
2. **Task 3:** the Office warning fired below the 60% floor too; it now says "He will not sign for that." Holdouts are also excluded from `toTeam`'s players (no pinch-hitting while holding out).
3. **Task 4:** (a) `aiWinter` must run after retirements and before the amateur fill (a buyer needs a vacancy) and the rivals shop the NEW winter pool (`genPool` moved ahead of it); a full buyer with a need cuts his least valuable spare to the pool. (b) The pool is a prospect market (mean 52), so tier-3 Boston never "needs" anyone; the test buyer is Baltimore. (c) The plan's `< 40000` net band was a guess; tier-3 clubs net ~$44–60k at 1918 scale (band widened to `< 80000`; tune 1 then grew salaries). (d) No-gate loop fixtures now fund the rivals, since broke rivals left the Reds no buyer. (e) Test fixtures must not set `L.phase = 'season'` before `simSeason` (story stops truncate the season).
4. **Task 5:** Rhode Island must never be in `makeSchedule`'s closed list: a closed Rhode Island plays its Sundays at Rocky Point (that is the M3 story), so the Reds keep their Sunday home dates. Closed-versus-closed Sunday fixtures keep their date (plan exception). The thirteen-season calibration test's own 30 s cap was lifted to the config's 60 s (M4 made the sim slower: 13 seasons ≈ 21 s alone, 31 s under the suite). Pre-existing: the calendar has no Sundays in 1923 and 1928.
5. **Task 6:** `check.ts` seeds its flag set with the engine-set `pennant_last` so `pennant-winter` passes the checker. The live-game fixtures in the clinch tests complete the Reds game through `closeLive`, not by clearing `L.live`.
6. **Task 7:** the 1920 letter already carried the lively-ball line. GOTCHA: Vitest serves `import.meta.glob` in `src/story/content.ts` from a cached transform, so new files under `content/` do not load under test until that module changes; a comment was added to the module. (`npm run build` saw the cards all along.)
7. **Task 8:** `sellPlayer(L, clubId, playerId, ...)` (the brief had the arguments reversed); standings are read after `toWalk` (history is written in `toWinter` and never for 1930); league RPG divides by two team-games; `node:fs` needs a narrow `@ts-expect-error` (no `@types/node`). First run: every bot ended 1930 with $150k–$585k and 35–50 wins.
8. **Task 9, tune 1:** salaries ride the boom (`salaryFor(p, year)` = base × `boom(year)`, sticky in 1929–30) and the bots shop in winter; that killed the cash explosion but most bots then ran deficits on the rescue loan. **Tune 2:** tier spread 3 → 2, `AI_MAX_BUYS` 2 → 1, bot reserves by policy (money-first $15,000). One finding left: Boston 8 of 13 under roots/m4-b (five under the other bots on the same seed) → the rival cap was 8 for one commit. **Tune 3** (after the review's accounting fix moved the dynasty to the gouger run at 9): tier spread 2 → 1; top rival 7 on every run; cap back at 7.
9. **Task 10:** Codex review found 0 Critical, 6 Important, 2 Minor; all fixed (`docs/m4-build/final-review.md` in the game repo). Final: b2a6f04, 482 tests green including the bots.
