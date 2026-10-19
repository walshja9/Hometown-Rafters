# M1 Baseball Core Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** One complete baseball game, simmed or played pitch by pitch on a bare diamond, using 1918–1930 run environments. It scores correctly, saves and reloads mid at-bat, and has its rule tests.

**Architecture:** A pure-TypeScript engine in `src/baseball/` (no DOM). Each plate appearance gets target outcome odds from ratings, through odds-ratio scaling plus log5 against the season's environment. A count-transition chain, solved per matchup, plays the pitches so that strikeouts, walks and HBP hit those targets exactly in expectation. Every mode (sim, managed, Moment) goes through one `applyAction(state, action)` function. Game state is plain JSON, including the RNG stream states, so saves are `JSON.stringify`. A minimal React page renders the diamond.

**Tech Stack:** Node 24, TypeScript (strict), Vite, React 19, Vitest. Data: Chadwick Baseball Databank (Lahman), CC BY-SA 3.0.

**Spec:** `docs/superpowers/specs/2026-10-19-era1-playable-slice-design.md` §9 (engine), §11 (architecture), §13 (M1).

## Global Constraints

- Game repo: `D:\CodexWorkspaces\hometown-rafters-game` (Git Bash: `/d/CodexWorkspaces/hometown-rafters-game`). Private GitHub remote `walshja9/hometown-rafters-game`.
- Fable implements directly. No Codex (Alex, 10/19).
- Runtime dependencies: `react`, `react-dom` only. Dev dependencies: `vite`, `@vitejs/plugin-react`, `typescript`, `vitest`, `@types/react`, `@types/react-dom`. Add nothing else.
- `src/baseball/`, `src/core/` and `src/save/` never import from `react`, `src/ui/`, or any DOM API.
- Ratings are 0–99; 65 is league average.
- One pitch-level engine for every mode. Sim mode is the same code path with AI-chosen actions.
- Randomness is seeded, with separate streams: `baseball` resolves pitches and plays; `manager` is used only by the AI manager. Viewing state never draws from any stream.
- Game state is plain JSON (no classes, Maps or functions) and carries `version: 1`.
- Data source pinned to commit `8a908d51833ee99b56fce165c93dc99ee8886d6f` of `github.com/cbwinslow/baseballdatabank` (a mirror of the Chadwick Bureau Databank). Commit only the derived per-season aggregates (`src/baseball/eras.json`), never the raw CSVs. Credit the source in README under CC BY-SA 3.0.
- Commit messages end with `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`. Stage explicit paths; never `git add -A`.
- Shell is Git Bash on Windows.

## Deferred from M1 (on purpose)

The hit-and-run, pitcher-side Moment actions, "sit on a pitch type", outs on the bases during extra-base attempts, the official-scorer hit-reduction rule on walk-offs, double switches, and darkness ties. The M3 plan picks up whichever of these the managed games and Moments need.

## File map

```
hometown-rafters-game/
  package.json, tsconfig.json, vite.config.ts, index.html, .gitignore, README.md
  scripts/build-eras.mjs        downloads pinned CSVs → src/baseball/eras.json
  src/core/rng.ts               seeded sfc32 RNG with JSON state; named streams
  src/baseball/types.ts         ratings, players, teams, game state, events, actions
  src/baseball/eras.json        generated: one record per season 1918–1930
  src/baseball/eras.ts          typed accessor getEra(year)
  src/baseball/fixtures.ts      deterministic test/demo teams
  src/baseball/matchup.ts       ratings + era → plate-appearance target odds
  src/baseball/pitchModel.ts    count chain, solver, per-pitch sampling, batter actions
  src/baseball/bases.ts         runner movement for every play outcome
  src/baseball/game.ts          newGame, legalActions, applyAction (the state machine)
  src/baseball/manager.ts       AI manager: aiAction(state)
  src/baseball/sim.ts           simulateGame(home, away, year, seed)
  src/baseball/boxScore.ts      fold events → box score
  src/save/save.ts              serialize / deserialize with version check
  src/ui/main.tsx, App.tsx, Diamond.tsx, styles.css
  tests/*.test.ts               one file per module, plus calibration
```

---

### Task 1: Scaffold the repo

**Files:**
- Create: `package.json`, `tsconfig.json`, `vite.config.ts`, `index.html`, `.gitignore`, `README.md`, `src/ui/main.tsx`, `src/ui/App.tsx`, `tests/smoke.test.ts`

**Interfaces:**
- Produces: `npm test` (Vitest, node environment), `npm run typecheck`, `npm run build`, `npm run dev`.

- [ ] **Step 1: Create the folder and package.json**

```bash
mkdir -p /d/CodexWorkspaces/hometown-rafters-game && cd /d/CodexWorkspaces/hometown-rafters-game && git init -b main
```

`package.json`:
```json
{
  "name": "hometown-rafters-game",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "typecheck": "tsc --noEmit",
    "test": "vitest run",
    "eras": "node scripts/build-eras.mjs"
  }
}
```

- [ ] **Step 2: Install dependencies**

```bash
cd /d/CodexWorkspaces/hometown-rafters-game && npm install react react-dom && npm install -D vite @vitejs/plugin-react typescript vitest @types/react @types/react-dom
```
Expected: installs without errors; `package.json` gains `dependencies` and `devDependencies`.

- [ ] **Step 3: Config files**

`tsconfig.json`:
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM"],
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "jsx": "react-jsx",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "skipLibCheck": true,
    "noEmit": true,
    "types": ["vitest/globals"]
  },
  "include": ["src", "tests"]
}
```

`vite.config.ts`:
```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: { environment: 'node', globals: true, testTimeout: 60_000 },
});
```

`index.html`:
```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Hometown Rafters</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/ui/main.tsx"></script>
  </body>
</html>
```

`.gitignore`:
```
node_modules/
dist/
.cache/
```

`src/ui/main.tsx`:
```tsx
import { createRoot } from 'react-dom/client';
import { App } from './App';

createRoot(document.getElementById('root')!).render(<App />);
```

`src/ui/App.tsx` (placeholder until Task 13):
```tsx
export function App() {
  return <h1>Hometown Rafters</h1>;
}
```

`README.md`:
```markdown
# Hometown Rafters (game)

The playable Era 1 slice of Hometown Rafters. Design: the Hometown Rafters docs repo,
`docs/superpowers/specs/2026-10-19-era1-playable-slice-design.md`.

    npm install
    npm test          # engine tests
    npm run dev       # bare diamond in the browser
    npm run eras      # rebuild src/baseball/eras.json from the pinned data

## Data credit

Season environments are derived from the Chadwick Baseball Databank
(github.com/chadwickbureau/baseballdatabank, via the cbwinslow mirror at commit 8a908d5),
licensed CC BY-SA 3.0. Only per-season league aggregates are stored in this repo.
```

- [ ] **Step 4: Smoke test**

`tests/smoke.test.ts`:
```ts
test('test runner works', () => {
  expect(1 + 1).toBe(2);
});
```

Run: `npm test && npm run typecheck`
Expected: 1 test passes; typecheck exits 0.

- [ ] **Step 5: Private remote and first commit**

```bash
cd /d/CodexWorkspaces/hometown-rafters-game
git add package.json package-lock.json tsconfig.json vite.config.ts index.html .gitignore README.md src/ui/main.tsx src/ui/App.tsx tests/smoke.test.ts
git commit -m "chore: scaffold Vite + React + TypeScript + Vitest" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
gh repo create walshja9/hometown-rafters-game --private --source . --push
```
Expected: the repo exists on GitHub as private; `git status -sb` shows `main...origin/main`.

---

### Task 2: Seeded RNG with JSON state

**Files:**
- Create: `src/core/rng.ts`
- Test: `tests/rng.test.ts`

**Interfaces:**
- Produces:
  - `type RngState = { a: number; b: number; c: number; d: number }`
  - `seedState(seed: string): RngState`
  - `next(s: RngState): number` (mutates `s`, returns [0, 1))
  - `type Streams = { baseball: RngState; manager: RngState }`
  - `makeStreams(seed: string): Streams`

- [ ] **Step 1: Write the failing tests**

`tests/rng.test.ts`:
```ts
import { seedState, next, makeStreams } from '../src/core/rng';

test('same seed gives the same sequence', () => {
  const a = seedState('reds-1918');
  const b = seedState('reds-1918');
  for (let i = 0; i < 100; i++) expect(next(a)).toBe(next(b));
});

test('different seeds differ', () => {
  const a = seedState('a');
  const b = seedState('b');
  expect(next(a)).not.toBe(next(b));
});

test('values are in [0, 1) and roughly uniform', () => {
  const s = seedState('uniform');
  let sum = 0;
  for (let i = 0; i < 100_000; i++) {
    const x = next(s);
    expect(x).toBeGreaterThanOrEqual(0);
    expect(x).toBeLessThan(1);
    sum += x;
  }
  expect(sum / 100_000).toBeCloseTo(0.5, 2);
});

test('state survives a JSON round trip mid-sequence', () => {
  const s = seedState('save');
  next(s); next(s);
  const copy = JSON.parse(JSON.stringify(s));
  for (let i = 0; i < 50; i++) expect(next(copy)).toBe(next(s));
});

test('streams are independent', () => {
  const s = makeStreams('game');
  const before = { ...s.baseball };
  next(s.manager);
  expect(s.baseball).toEqual(before);
});
```

- [ ] **Step 2: Run to verify failure**

Run: `npx vitest run tests/rng.test.ts`
Expected: FAIL (cannot resolve `../src/core/rng`).

- [ ] **Step 3: Implement**

`src/core/rng.ts`:
```ts
// sfc32 (small fast counter) with xmur3 seeding. State is four uint32s in a plain object,
// so it serializes with JSON and a reloaded save continues the exact sequence.
export type RngState = { a: number; b: number; c: number; d: number };
export type Streams = { baseball: RngState; manager: RngState };

function xmur3(str: string): () => number {
  let h = 1779033703 ^ str.length;
  for (let i = 0; i < str.length; i++) {
    h = Math.imul(h ^ str.charCodeAt(i), 3432918353);
    h = (h << 13) | (h >>> 19);
  }
  return () => {
    h = Math.imul(h ^ (h >>> 16), 2246822507);
    h = Math.imul(h ^ (h >>> 13), 3266489909);
    return (h ^= h >>> 16) >>> 0;
  };
}

export function seedState(seed: string): RngState {
  const h = xmur3(seed);
  const s = { a: h(), b: h(), c: h(), d: h() };
  for (let i = 0; i < 12; i++) next(s); // warm up
  return s;
}

export function next(s: RngState): number {
  const t = (((s.a + s.b) | 0) + s.d) | 0;
  s.d = (s.d + 1) | 0;
  s.a = s.b ^ (s.b >>> 9);
  s.b = (s.c + (s.c << 3)) | 0;
  s.c = (s.c << 21) | (s.c >>> 11);
  s.c = (s.c + t) | 0;
  return (t >>> 0) / 4294967296;
}

export function makeStreams(seed: string): Streams {
  return { baseball: seedState(`${seed}:baseball`), manager: seedState(`${seed}:manager`) };
}
```

- [ ] **Step 4: Run tests**

Run: `npx vitest run tests/rng.test.ts`
Expected: 5 pass.

- [ ] **Step 5: Commit**

```bash
git add src/core/rng.ts tests/rng.test.ts
git commit -m "feat: seeded sfc32 RNG with JSON state and named streams" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 3: Season environments from real data

**Files:**
- Create: `scripts/build-eras.mjs`, `src/baseball/eras.ts`, `src/baseball/eras.json` (generated)
- Test: `tests/eras.test.ts`

**Interfaces:**
- Produces:
  - `interface Era { year; k; bb; hbp; hrPerContact; babip; doubleShare; tripleShare; errorsPerGame; roePerBip; sbPerGame; csRate; shPerGame; runsPerGame }` (all `number`)
  - `getEra(year: number): Era` (throws outside 1918–1930)
  - `ERA_YEARS: number[]`

Facts checked while writing this plan (10/19): the pinned CSVs parse with a plain comma split for 1918–1930 (no quoted fields; Teams has 48 columns and Batting has 22, every row). `Teams.csv` HBP is empty for all of 1918–1930 and CS is empty for 72 of 208 team-seasons, so HBP and SH come from `Batting.csv`, and the CS rate uses only team-seasons that record CS. A preview: 1918 had K 7.8%, BB 7.6%, HR/PA 0.31% and BABIP .276; 1930 had K 8.2%, HR/PA 1.61% and BABIP .312.

- [ ] **Step 1: Write the build script**

`scripts/build-eras.mjs`:
```js
// Builds src/baseball/eras.json: one league-wide (AL+NL) environment per season, 1918–1930.
// Source: Chadwick Baseball Databank, CC BY-SA 3.0, pinned mirror commit.
import { writeFileSync } from 'node:fs';

const SHA = '8a908d51833ee99b56fce165c93dc99ee8886d6f';
const BASE = `https://raw.githubusercontent.com/cbwinslow/baseballdatabank/${SHA}/core`;
const FIRST = 1918;
const LAST = 1930;
// Share of fielding errors that put the batter on base. Not in the data; an assumption the
// calibration test (Task 12) checks through runs per game.
const ROE_SHARE_OF_ERRORS = 0.5;
const CS_RATE_FALLBACK = 0.45;

async function csv(name) {
  const res = await fetch(`${BASE}/${name}`);
  if (!res.ok) throw new Error(`${name}: HTTP ${res.status}`);
  const [header, ...lines] = (await res.text()).trim().split(/\r?\n/);
  const cols = header.split(',');
  return lines.map((line) => {
    const v = line.split(',');
    return Object.fromEntries(cols.map((c, i) => [c, v[i] ?? '']));
  });
}

const num = (x) => (x === '' ? 0 : Number(x));

const [teams, batting] = await Promise.all([csv('Teams.csv'), csv('Batting.csv')]);
const eras = [];
for (let year = FIRST; year <= LAST; year++) {
  const ty = teams.filter((t) => num(t.yearID) === year && (t.lgID === 'AL' || t.lgID === 'NL'));
  const by = batting.filter((b) => num(b.yearID) === year && (b.lgID === 'AL' || b.lgID === 'NL'));
  const sum = (rows, col) => rows.reduce((acc, r) => acc + num(r[col]), 0);
  const AB = sum(by, 'AB'), H = sum(by, 'H'), D = sum(by, '2B'), T = sum(by, '3B'), HR = sum(by, 'HR');
  const BB = sum(by, 'BB'), SO = sum(by, 'SO'), HBP = sum(by, 'HBP'), SH = sum(by, 'SH');
  const PA = AB + BB + HBP + SH;
  const G = sum(ty, 'G'), R = sum(ty, 'R'), E = sum(ty, 'E'), SB = sum(ty, 'SB');
  const withCs = ty.filter((t) => t.CS !== '');
  const csSb = sum(withCs, 'SB'), cs = sum(withCs, 'CS');
  const bip = AB - SO - HR;
  eras.push({
    year,
    k: SO / PA,
    bb: BB / PA,
    hbp: HBP / PA,
    hrPerContact: HR / (AB - SO),
    babip: (H - HR) / bip,
    doubleShare: D / (H - HR),
    tripleShare: T / (H - HR),
    errorsPerGame: E / G,
    roePerBip: (ROE_SHARE_OF_ERRORS * E) / bip,
    sbPerGame: SB / G,
    csRate: cs + csSb > 0 ? cs / (cs + csSb) : CS_RATE_FALLBACK,
    shPerGame: SH / G,
    runsPerGame: R / G,
  });
}
const round = (x) => Math.round(x * 1e6) / 1e6;
const out = eras.map((e) => Object.fromEntries(Object.entries(e).map(([k, v]) => [k, k === 'year' ? v : round(v)])));
writeFileSync(new URL('../src/baseball/eras.json', import.meta.url), JSON.stringify(out, null, 2) + '\n');
console.log(`wrote ${out.length} seasons`, out.map((e) => `${e.year}:${e.runsPerGame.toFixed(2)}R/G`).join(' '));
```

- [ ] **Step 2: Generate the data**

Run: `npm run eras`
Expected: `wrote 13 seasons` followed by runs per game for each year, roughly 3.6 in 1918 rising to about 5.5 in 1930.

- [ ] **Step 3: Write the failing tests**

`tests/eras.test.ts`:
```ts
import { getEra, ERA_YEARS } from '../src/baseball/eras';

test('covers 1918 through 1930', () => {
  expect(ERA_YEARS).toEqual([1918, 1919, 1920, 1921, 1922, 1923, 1924, 1925, 1926, 1927, 1928, 1929, 1930]);
});

test('rates are in plausible ranges every season', () => {
  for (const y of ERA_YEARS) {
    const e = getEra(y);
    expect(e.k).toBeGreaterThan(0.04); expect(e.k).toBeLessThan(0.12);
    expect(e.bb).toBeGreaterThan(0.05); expect(e.bb).toBeLessThan(0.11);
    expect(e.hbp).toBeGreaterThan(0.002); expect(e.hbp).toBeLessThan(0.015);
    expect(e.babip).toBeGreaterThan(0.25); expect(e.babip).toBeLessThan(0.34);
    expect(e.runsPerGame).toBeGreaterThan(3); expect(e.runsPerGame).toBeLessThan(6.5);
    expect(e.csRate).toBeGreaterThan(0.2); expect(e.csRate).toBeLessThan(0.7);
  }
});

test('the live-ball shift shows up', () => {
  expect(getEra(1930).hrPerContact).toBeGreaterThan(getEra(1918).hrPerContact * 3);
});

test('years outside the slice throw', () => {
  expect(() => getEra(1931)).toThrow();
});
```

- [ ] **Step 4: Run to verify failure**

Run: `npx vitest run tests/eras.test.ts`
Expected: FAIL (cannot resolve `../src/baseball/eras`).

- [ ] **Step 5: Implement the accessor**

`src/baseball/eras.ts`:
```ts
import data from './eras.json';

export interface Era {
  year: number;
  k: number; bb: number; hbp: number; // per plate appearance
  hrPerContact: number;               // HR / (AB - SO)
  babip: number;                      // (H - HR) / (AB - SO - HR)
  doubleShare: number; tripleShare: number; // shares of non-HR hits
  errorsPerGame: number;              // per team-game
  roePerBip: number;                  // batter reaches on error, per ball in play
  sbPerGame: number; csRate: number;  // per team-game; CS / (SB + CS)
  shPerGame: number;                  // sacrifice hits per team-game
  runsPerGame: number;                // per team-game (calibration target)
}

const ERAS = data as Era[];
export const ERA_YEARS = ERAS.map((e) => e.year);

export function getEra(year: number): Era {
  const e = ERAS.find((x) => x.year === year);
  if (!e) throw new Error(`no environment for ${year}; the slice covers 1918–1930`);
  return e;
}
```

- [ ] **Step 6: Run tests**

Run: `npx vitest run tests/eras.test.ts && npm run typecheck`
Expected: 4 pass; typecheck exits 0. If a range assertion fails, inspect `eras.json` before changing anything: a failure means either the data or the script is wrong, and the test range was set from the 10/19 preview.

- [ ] **Step 7: Commit**

```bash
git add scripts/build-eras.mjs src/baseball/eras.ts src/baseball/eras.json tests/eras.test.ts
git commit -m "feat: 1918-1930 season environments from the Databank" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 4: Engine types and fixture teams

**Files:**
- Create: `src/baseball/types.ts`, `src/baseball/fixtures.ts`
- Test: `tests/fixtures.test.ts`

**Interfaces:**
- Consumes: `Streams` from `src/core/rng.ts`.
- Produces: every type below (used by all later tasks), plus `avgBatting`, `avgPitching`, `PITCHER_BATTING`, `makeTeam(id, name, opts?)`.

- [ ] **Step 1: Write the types**

`src/baseball/types.ts`:
```ts
import type { Streams } from '../core/rng';

export type Hand = 'L' | 'R';
export type Position = 'P' | 'C' | '1B' | '2B' | '3B' | 'SS' | 'LF' | 'CF' | 'RF';
export type Side = 'away' | 'home';
export type Half = 'top' | 'bottom';

// All ratings 0–99; 65 is league average.
export interface Batting { contact: number; power: number; eye: number; avoidK: number; speed: number; fielding: number; arm: number }
export interface Pitching { control: number; stuff: number; contactSuppression: number; stamina: number }
export interface Player { id: string; name: string; bats: Hand; throws: Hand; batting: Batting; pitching?: Pitching }

export interface LineupSlot { playerId: string; pos: Position }
// lineup: 9 slots in batting order; the slot with pos 'P' holds startingPitcherId.
export interface Team { id: string; name: string; players: Player[]; lineup: LineupSlot[]; startingPitcherId: string }

export interface Runner { playerId: string; pitcherId: string; onError: boolean } // pitcherId = pitcher charged if he scores
export type Bases = [Runner | null, Runner | null, Runner | null];

export interface SideState {
  lineup: LineupSlot[];   // current order, after substitutions
  battingIndex: number;   // next batter, 0..8
  pitcherId: string;      // this side's current pitcher
  pitchCount: number;     // current pitcher's pitches
  runsAllowed: number;    // current pitcher's runs allowed
  removed: string[];      // players out of the game; they cannot re-enter
  needsPitcher: boolean;  // the pitcher's slot was pinch-hit; must change pitchers before pitching again
}

export type PitchResult = 'ball' | 'strike' | 'foul' | 'hbp' | 'inPlay';
export type PaOutcome =
  | 'K' | 'BB' | 'IBB' | 'HBP' | '1B' | '2B' | '3B' | 'HR' | 'ROE'
  | 'GO' | 'GDP' | 'FC' | 'FO' | 'SF' | 'LO' | 'SAC' | 'BUNT_HIT';

export type GameEvent =
  | { t: 'pitch'; result: PitchResult; batterId: string; pitcherId: string; balls: number; strikes: number }
  | { t: 'pa'; outcome: PaOutcome; batterId: string; pitcherId: string; outs: number; rbi: number; side: Side }
  | { t: 'run'; runnerId: string; pitcherId: string; earned: boolean; side: Side }
  | { t: 'steal'; runnerId: string; pitcherId: string; from: 1 | 2; success: boolean; side: Side }
  | { t: 'sub'; kind: 'PH' | 'P'; inId: string; outId: string; side: Side }
  | { t: 'half'; inning: number; half: Half }
  | { t: 'final'; away: number; home: number; innings: number };

export type BatterAction = 'normal' | 'take' | 'protect' | 'bunt';
export type Action =
  | { kind: 'pitch'; batter: BatterAction }
  | { kind: 'steal'; from: 1 | 2 }
  | { kind: 'ibb' }
  | { kind: 'pinchHit'; inId: string }
  | { kind: 'changePitcher'; inId: string };

export interface GameState {
  version: 1;
  year: number;
  teams: Record<Side, Team>;
  sides: Record<Side, SideState>;
  inning: number;
  half: Half;
  outs: number;
  balls: number;
  strikes: number;
  bases: Bases;
  score: Record<Side, number>;
  lineScore: Record<Side, number[]>;
  rng: Streams;
  events: GameEvent[];
  final: boolean;
}
```

- [ ] **Step 2: Write the failing fixture test**

`tests/fixtures.test.ts`:
```ts
import { makeTeam } from '../src/baseball/fixtures';

test('makeTeam builds a legal 17-man team', () => {
  const t = makeTeam('aw', 'Away');
  expect(t.lineup).toHaveLength(9);
  expect(t.players).toHaveLength(17);
  expect(new Set(t.players.map((p) => p.id)).size).toBe(17);
  const pSlots = t.lineup.filter((s) => s.pos === 'P');
  expect(pSlots).toHaveLength(1);
  expect(pSlots[0]!.playerId).toBe(t.startingPitcherId);
  expect(t.players.filter((p) => p.pitching)).toHaveLength(5);
  expect(new Set(t.lineup.map((s) => s.pos)).size).toBe(9);
});

test('overrides apply to every hitter or pitcher', () => {
  const t = makeTeam('x', 'X', { batting: { power: 90 }, pitching: { stuff: 80 } });
  expect(t.players.filter((p) => !p.pitching).every((p) => p.batting.power === 90)).toBe(true);
  expect(t.players.filter((p) => p.pitching).every((p) => p.pitching!.stuff === 80)).toBe(true);
});
```

- [ ] **Step 3: Run to verify failure**

Run: `npx vitest run tests/fixtures.test.ts`
Expected: FAIL (cannot resolve `../src/baseball/fixtures`).

- [ ] **Step 4: Implement fixtures**

`src/baseball/fixtures.ts`:
```ts
import type { Batting, Pitching, Player, Position, Team, LineupSlot } from './types';

export function avgBatting(over: Partial<Batting> = {}): Batting {
  return { contact: 65, power: 65, eye: 65, avoidK: 65, speed: 65, fielding: 65, arm: 65, ...over };
}
export function avgPitching(over: Partial<Pitching> = {}): Pitching {
  return { control: 65, stuff: 65, contactSuppression: 65, stamina: 65, ...over };
}
// Pitchers hit like pitchers did: badly.
export const PITCHER_BATTING: Batting = { contact: 35, power: 25, eye: 35, avoidK: 35, speed: 45, fielding: 60, arm: 65 };

const ORDER: Position[] = ['CF', '2B', 'LF', '1B', 'RF', '3B', 'SS', 'C'];

// 8 position starters + SP batting 9th, 4 bench bats, 4 relievers = 17 players.
// ponytail: engine-test fixture only; M2 generates real 25-man rosters.
export function makeTeam(id: string, name: string, opts: { batting?: Partial<Batting>; pitching?: Partial<Pitching> } = {}): Team {
  const players: Player[] = [];
  const lineup: LineupSlot[] = [];
  ORDER.forEach((pos, i) => {
    const p: Player = { id: `${id}-b${i + 1}`, name: `${name} ${pos}`, bats: i % 3 === 0 ? 'L' : 'R', throws: 'R', batting: avgBatting(opts.batting) };
    players.push(p);
    lineup.push({ playerId: p.id, pos });
  });
  const sp: Player = { id: `${id}-p1`, name: `${name} SP`, bats: 'R', throws: 'R', batting: PITCHER_BATTING, pitching: avgPitching(opts.pitching) };
  players.push(sp);
  lineup.push({ playerId: sp.id, pos: 'P' });
  for (let i = 1; i <= 4; i++) {
    players.push({ id: `${id}-bench${i}`, name: `${name} Bench ${i}`, bats: 'R', throws: 'R', batting: avgBatting(opts.batting) });
  }
  for (let i = 2; i <= 5; i++) {
    players.push({ id: `${id}-p${i}`, name: `${name} RP${i - 1}`, bats: 'R', throws: i % 2 ? 'L' : 'R', batting: PITCHER_BATTING, pitching: avgPitching(opts.pitching) });
  }
  return { id, name, players, lineup, startingPitcherId: sp.id };
}
```

- [ ] **Step 5: Run tests**

Run: `npx vitest run tests/fixtures.test.ts && npm run typecheck`
Expected: 2 pass; typecheck exits 0.

- [ ] **Step 6: Commit**

```bash
git add src/baseball/types.ts src/baseball/fixtures.ts tests/fixtures.test.ts
git commit -m "feat: engine types and fixture teams" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 5: Plate-appearance target odds

**Files:**
- Create: `src/baseball/matchup.ts`
- Test: `tests/matchup.test.ts`

**Interfaces:**
- Consumes: `Batting`, `Pitching`, `Hand` (Task 4); `Era`, `getEra`, `ERA_YEARS` (Task 3).
- Produces:
  - `type PaKey = 'K'|'BB'|'HBP'|'HR'|'3B'|'2B'|'1B'|'ROE'|'GO'|'FO'|'LO'`
  - `type PaTargets = Record<PaKey, number>` (sums to 1)
  - `interface MatchupInput { batter: Batting; bats: Hand; pitcher: Pitching; throws: Hand; fatigue: number; defenseFielding: number; era: Era }`
  - `paTargets(m: MatchupInput): PaTargets`
  - `scaleRate(lg, rating, k)`, `log5(b, p, lg)`

This replaces `sim_engine.py`'s linear rating-to-rate slopes with odds-ratio scaling. Every 10 rating points multiplies the odds by `e^k`, so a rate stays inside (0, 1) for any rating in any era. The old linear slopes were sized for a 22.5% strikeout league; at 1918's 7.8% they would go negative. The strikeout, walk and HBP sum is capped at 0.9, which fixes the old bug where independent odds could exceed 1.

- [ ] **Step 1: Write the failing tests**

`tests/matchup.test.ts`:
```ts
import { paTargets, type MatchupInput, type PaKey } from '../src/baseball/matchup';
import { getEra, ERA_YEARS } from '../src/baseball/eras';
import { avgBatting, avgPitching } from '../src/baseball/fixtures';

const KEYS: PaKey[] = ['K', 'BB', 'HBP', 'HR', '3B', '2B', '1B', 'ROE', 'GO', 'FO', 'LO'];
const base = (over: Partial<MatchupInput> = {}): MatchupInput => ({
  batter: avgBatting(), bats: 'R', pitcher: avgPitching(), throws: 'R',
  fatigue: 0, defenseFielding: 65, era: getEra(1918), ...over,
});
const sum = (t: Record<string, number>) => Object.values(t).reduce((a, b) => a + b, 0);

test('odds sum to 1 and stay non-negative at every extreme, every season', () => {
  const ratings = [0, 65, 99];
  for (const y of ERA_YEARS) for (const r of ratings) for (const q of ratings) for (const fatigue of [0, 1]) {
    const all = (v: number) => ({ contact: v, power: v, eye: v, avoidK: v, speed: v, fielding: v, arm: v });
    const t = paTargets(base({
      era: getEra(y), fatigue, defenseFielding: q,
      batter: all(r), pitcher: { control: q, stuff: q, contactSuppression: q, stamina: q },
    }));
    expect(Math.abs(sum(t) - 1)).toBeLessThan(1e-9);
    for (const k of KEYS) expect(t[k]).toBeGreaterThanOrEqual(0);
  }
});

test('average hitter vs average pitcher reproduces the season K and BB rates', () => {
  for (const y of ERA_YEARS) {
    const t = paTargets(base({ era: getEra(y) }));
    expect(t.K).toBeCloseTo(getEra(y).k, 9);
    expect(t.BB).toBeCloseTo(getEra(y).bb, 9);
  }
});

test('ratings push outcomes the right way', () => {
  const t0 = paTargets(base());
  expect(paTargets(base({ batter: avgBatting({ avoidK: 90 }) })).K).toBeLessThan(t0.K);
  expect(paTargets(base({ pitcher: avgPitching({ stuff: 90 }) })).K).toBeGreaterThan(t0.K);
  expect(paTargets(base({ batter: avgBatting({ eye: 90 }) })).BB).toBeGreaterThan(t0.BB);
  expect(paTargets(base({ pitcher: avgPitching({ control: 90 }) })).BB).toBeLessThan(t0.BB);
  expect(paTargets(base({ batter: avgBatting({ power: 90 }) })).HR).toBeGreaterThan(t0.HR);
  expect(paTargets(base({ defenseFielding: 90 })).ROE).toBeLessThan(t0.ROE);
});

test('a tired pitcher walks more and strikes out fewer', () => {
  const fresh = paTargets(base());
  const tired = paTargets(base({ fatigue: 1 }));
  expect(tired.BB).toBeGreaterThan(fresh.BB);
  expect(tired.K).toBeLessThan(fresh.K);
});

test('opposite-hand matchups favor the hitter', () => {
  const same = paTargets(base({ bats: 'R', throws: 'R' }));
  const opp = paTargets(base({ bats: 'L', throws: 'R' }));
  expect(opp['1B'] + opp['2B'] + opp['3B'] + opp.HR).toBeGreaterThan(same['1B'] + same['2B'] + same['3B'] + same.HR);
});
```

- [ ] **Step 2: Run to verify failure**

Run: `npx vitest run tests/matchup.test.ts`
Expected: FAIL (cannot resolve `../src/baseball/matchup`).

- [ ] **Step 3: Implement**

`src/baseball/matchup.ts`:
```ts
import type { Batting, Hand, Pitching } from './types';
import type { Era } from './eras';

export type PaKey = 'K' | 'BB' | 'HBP' | 'HR' | '3B' | '2B' | '1B' | 'ROE' | 'GO' | 'FO' | 'LO';
export type PaTargets = Record<PaKey, number>;
export interface MatchupInput {
  batter: Batting; bats: Hand;
  pitcher: Pitching; throws: Hand;
  fatigue: number;          // 0 fresh .. 1 spent
  defenseFielding: number;  // average fielding rating of the eight fielders behind the pitcher
  era: Era;
}

const ANCHOR = 65;
const PLATOON = 3;      // rating points of contact and power, + opposite hand / - same hand
const TIRED = 15;       // rating points a fully spent pitcher loses on every skill
const WALL_CAP = 0.9;   // K + BB + HBP never exceed this
const OUT_SPLIT = { GO: 0.48, FO: 0.36, LO: 0.16 }; // ponytail: era-constant out mix; tune if calibration needs it

const odds = (p: number) => p / (1 - p);
const prob = (o: number) => o / (1 + o);

export function scaleRate(lg: number, rating: number, k: number): number {
  return prob(odds(lg) * Math.exp((k * (rating - ANCHOR)) / 10));
}

export function log5(b: number, p: number, lg: number): number {
  const n = (b * p) / lg;
  return n / (n + ((1 - b) * (1 - p)) / (1 - lg));
}

export function paTargets(m: MatchupInput): PaTargets {
  const e = m.era;
  const plat = m.bats === m.throws ? -PLATOON : PLATOON;
  const t = TIRED * m.fatigue;
  const stuff = m.pitcher.stuff - t;
  const control = m.pitcher.control - t;
  const supp = m.pitcher.contactSuppression - t;
  const contact = m.batter.contact + plat;
  const power = m.batter.power + plat;

  const k = log5(scaleRate(e.k, m.batter.avoidK, -0.25), scaleRate(e.k, stuff, 0.25), e.k);
  const bb = log5(scaleRate(e.bb, m.batter.eye, 0.2), scaleRate(e.bb, control, -0.2), e.bb);
  const walls = k + bb + e.hbp;
  const shrink = walls > WALL_CAP ? WALL_CAP / walls : 1;
  const pK = k * shrink, pBB = bb * shrink, pHBP = e.hbp * shrink;

  const contacted = 1 - pK - pBB - pHBP;
  const hrpc = log5(scaleRate(e.hrPerContact, power, 0.35), scaleRate(e.hrPerContact, supp, -0.25), e.hrPerContact);
  const pHR = contacted * hrpc;
  const bip = contacted - pHR;

  const babip = scaleRate(log5(scaleRate(e.babip, contact, 0.08), scaleRate(e.babip, supp, -0.08), e.babip), m.defenseFielding, -0.05);
  const roe = Math.min(scaleRate(e.roePerBip, m.defenseFielding, -0.3), 0.5 * (1 - babip));
  const hits = bip * babip;
  const pROE = bip * roe;
  const outs = bip - hits - pROE;

  let d = e.doubleShare * Math.exp((0.1 * (power - ANCHOR)) / 10);
  let tr = e.tripleShare * Math.exp((0.15 * (m.batter.speed - ANCHOR)) / 10);
  if (d + tr > 0.8) { const s = 0.8 / (d + tr); d *= s; tr *= s; }

  const fo = OUT_SPLIT.FO * Math.exp((0.1 * (power - ANCHOR)) / 10);
  const w = OUT_SPLIT.GO + fo + OUT_SPLIT.LO;
  return {
    K: pK, BB: pBB, HBP: pHBP, HR: pHR,
    '3B': hits * tr, '2B': hits * d, '1B': hits * (1 - d - tr), ROE: pROE,
    GO: (outs * OUT_SPLIT.GO) / w, FO: (outs * fo) / w, LO: (outs * OUT_SPLIT.LO) / w,
  };
}
```

- [ ] **Step 4: Run tests**

Run: `npx vitest run tests/matchup.test.ts && npm run typecheck`
Expected: 5 pass; typecheck exits 0.

- [ ] **Step 5: Commit**

```bash
git add src/baseball/matchup.ts tests/matchup.test.ts
git commit -m "feat: plate-appearance odds via odds-ratio scaling and log5" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 6: The pitch model (count chain, solver, batter actions)

**Files:**
- Create: `src/baseball/pitchModel.ts`
- Test: `tests/pitchModel.test.ts`

**Interfaces:**
- Consumes: `PaTargets` (Task 5); `BatterAction`, `PitchResult` (Task 4); `next`, `seedState` (Task 2).
- Produces:
  - `interface PitchProbs { ball; strike; foul; hbp; inPlay }` (all `number`, summing to 1)
  - `FOUL` (0.18), `CALLED_SHARE` (0.45)
  - `chainOutcomes(p: PitchProbs): { K: number; BB: number; HBP: number; IP: number }`
  - `solvePitchProbs(t: { K: number; BB: number; HBP: number }): PitchProbs` (cached)
  - `actionProbs(base: PitchProbs, action: BatterAction): PitchProbs`
  - `samplePitch(p: PitchProbs, r: number): PitchResult`
  - `type InPlayKey = 'HR'|'3B'|'2B'|'1B'|'ROE'|'GO'|'FO'|'LO'`
  - `inPlayWeights(t: PaTargets, action: BatterAction): Record<InPlayKey, number>`
  - `pick<K extends string>(w: Record<K, number>, r: number): K`

How it works: per-pitch probabilities are constant across counts. A foul with two strikes leaves the count unchanged. `chainOutcomes` computes the chance a plate appearance from 0-0 ends in K, BB, HBP or a ball in play. `solvePitchProbs` adjusts the ball, strike and HBP probabilities until those match the matchup's targets. The in-play result is then drawn from the matchup's in-play shares, so over many plate appearances every outcome hits its target. With `'normal'` actions, sim mode and pitch-by-pitch play are the same computation.

- [ ] **Step 1: Write the failing tests**

`tests/pitchModel.test.ts`:
```ts
import { chainOutcomes, solvePitchProbs, actionProbs, samplePitch, pick, FOUL, type PitchProbs } from '../src/baseball/pitchModel';
import { paTargets } from '../src/baseball/matchup';
import { getEra } from '../src/baseball/eras';
import { avgBatting, avgPitching } from '../src/baseball/fixtures';
import { seedState, next } from '../src/core/rng';

const total = (p: PitchProbs) => p.ball + p.strike + p.foul + p.hbp + p.inPlay;

function playPa(p: PitchProbs, rng: ReturnType<typeof seedState>): { end: 'K' | 'BB' | 'HBP' | 'IP'; pitches: number } {
  let b = 0, s = 0, n = 0;
  for (;;) {
    n++;
    const r = samplePitch(p, next(rng));
    if (r === 'ball') { if (++b === 4) return { end: 'BB', pitches: n }; }
    else if (r === 'strike') { if (++s === 3) return { end: 'K', pitches: n }; }
    else if (r === 'foul') { if (s < 2) s++; }
    else if (r === 'hbp') return { end: 'HBP', pitches: n };
    else return { end: 'IP', pitches: n };
  }
}

test('chain outcomes sum to 1', () => {
  const o = chainOutcomes({ ball: 0.35, strike: 0.25, foul: FOUL, hbp: 0.004, inPlay: 0.216 });
  expect(o.K + o.BB + o.HBP + o.IP).toBeCloseTo(1, 12);
});

test('solver hits the targets for average and extreme matchups', () => {
  const cases = [
    paTargets({ batter: avgBatting(), bats: 'R', pitcher: avgPitching(), throws: 'R', fatigue: 0, defenseFielding: 65, era: getEra(1918) }),
    paTargets({ batter: avgBatting(), bats: 'L', pitcher: avgPitching(), throws: 'R', fatigue: 0, defenseFielding: 65, era: getEra(1930) }),
    paTargets({ batter: avgBatting({ avoidK: 0, eye: 0 }), bats: 'R', pitcher: avgPitching({ stuff: 99, control: 0 }), throws: 'R', fatigue: 1, defenseFielding: 65, era: getEra(1925) }),
    paTargets({ batter: avgBatting({ avoidK: 99, eye: 99 }), bats: 'R', pitcher: avgPitching({ stuff: 0, control: 99 }), throws: 'R', fatigue: 0, defenseFielding: 65, era: getEra(1925) }),
  ];
  for (const t of cases) {
    const p = solvePitchProbs(t);
    expect(total(p)).toBeCloseTo(1, 12);
    for (const v of Object.values(p)) expect(v).toBeGreaterThanOrEqual(0);
    const o = chainOutcomes(p);
    expect(Math.abs(o.K - t.K)).toBeLessThan(1e-6);
    expect(Math.abs(o.BB - t.BB)).toBeLessThan(1e-6);
    expect(Math.abs(o.HBP - t.HBP)).toBeLessThan(1e-6);
  }
});

test('playing pitches reproduces the targets (Monte Carlo)', () => {
  const t = paTargets({ batter: avgBatting(), bats: 'R', pitcher: avgPitching(), throws: 'R', fatigue: 0, defenseFielding: 65, era: getEra(1918) });
  const p = solvePitchProbs(t);
  const rng = seedState('mc');
  const n = 200_000;
  const c = { K: 0, BB: 0, HBP: 0, IP: 0 };
  let pitches = 0;
  for (let i = 0; i < n; i++) { const r = playPa(p, rng); c[r.end]++; pitches += r.pitches; }
  expect(c.K / n).toBeCloseTo(t.K, 2);
  expect(c.BB / n).toBeCloseTo(t.BB, 2);
  expect(pitches / n).toBeGreaterThan(3);
  expect(pitches / n).toBeLessThan(4.2);
});

test('batter actions produce valid distributions with the expected tilt', () => {
  const base = solvePitchProbs({ K: 0.078, BB: 0.076, HBP: 0.006 });
  const take = actionProbs(base, 'take');
  expect(total(take)).toBeCloseTo(1, 12);
  expect(take.inPlay).toBe(0);
  expect(take.foul).toBe(0);
  const protect = actionProbs(base, 'protect');
  expect(total(protect)).toBeCloseTo(1, 12);
  expect(protect.foul).toBeGreaterThan(base.foul);
  const bunt = actionProbs(base, 'bunt');
  expect(total(bunt)).toBeCloseTo(1, 12);
  expect(bunt.inPlay).toBeGreaterThan(base.inPlay);
  expect(actionProbs(base, 'normal')).toBe(base);
});

test('pick follows cumulative weights', () => {
  const w = { a: 1, b: 3 };
  expect(pick(w, 0)).toBe('a');
  expect(pick(w, 0.24)).toBe('a');
  expect(pick(w, 0.26)).toBe('b');
  expect(pick(w, 0.999)).toBe('b');
});
```

- [ ] **Step 2: Run to verify failure**

Run: `npx vitest run tests/pitchModel.test.ts`
Expected: FAIL (cannot resolve `../src/baseball/pitchModel`).

- [ ] **Step 3: Implement**

`src/baseball/pitchModel.ts`:
```ts
import type { PaTargets } from './matchup';
import type { BatterAction, PitchResult } from './types';

export interface PitchProbs { ball: number; strike: number; foul: number; hbp: number; inPlay: number }
export type InPlayKey = 'HR' | '3B' | '2B' | '1B' | 'ROE' | 'GO' | 'FO' | 'LO';

export const FOUL = 0.18;         // ponytail: one era-wide foul rate; it mainly sets pitches per PA
export const CALLED_SHARE = 0.45; // share of strikes that are taken rather than swung through
const MIN_IN_PLAY = 0.02;

type Vec = [number, number, number, number]; // K, BB, HBP, IP

// P(PA ends in K/BB/HBP/IP) starting from 0-0, with constant per-pitch probabilities.
export function chainOutcomes(p: PitchProbs): { K: number; BB: number; HBP: number; IP: number } {
  const f: Vec[][] = Array.from({ length: 5 }, () => Array.from({ length: 3 }, () => [0, 0, 0, 0] as Vec));
  for (let s = 0; s < 3; s++) f[4]![s] = [0, 1, 0, 0]; // ball four
  const strikeOut = (b: number, s: number): Vec => (s === 2 ? [1, 0, 0, 0] : f[b]![s + 1]!);
  for (let s = 2; s >= 0; s--) {
    for (let b = 3; b >= 0; b--) {
      const ball = f[b + 1]![s]!;
      const strike = strikeOut(b, s);
      const v: Vec = [0, 0, 0, 0];
      for (let i = 0; i < 4; i++) {
        v[i] = p.ball * ball[i]! + p.strike * strike[i]! + (i === 2 ? p.hbp : 0) + (i === 3 ? p.inPlay : 0);
      }
      if (s < 2) {
        const foul = f[b]![s + 1]!;
        for (let i = 0; i < 4; i++) v[i] = v[i]! + p.foul * foul[i]!;
      } else {
        for (let i = 0; i < 4; i++) v[i] = v[i]! / (1 - p.foul); // two-strike fouls repeat the count
      }
      f[b]![s] = v;
    }
  }
  const [K, BB, HBP, IP] = f[0]![0]!;
  return { K, BB, HBP, IP };
}

const cache = new Map<string, PitchProbs>();

export function solvePitchProbs(t: { K: number; BB: number; HBP: number }): PitchProbs {
  const key = `${t.K.toFixed(7)}|${t.BB.toFixed(7)}|${t.HBP.toFixed(7)}`;
  const hit = cache.get(key);
  if (hit) return hit;
  let ball = 0.35, strike = 0.25, hbp = Math.max(t.HBP, 1e-9);
  const room = 1 - FOUL - MIN_IN_PLAY;
  for (let i = 0; i < 2000; i++) {
    const o = chainOutcomes({ ball, strike, foul: FOUL, hbp, inPlay: 1 - FOUL - ball - strike - hbp });
    const err = Math.max(Math.abs(o.K - t.K), Math.abs(o.BB - t.BB), Math.abs(o.HBP - t.HBP));
    if (err < 1e-10) break;
    ball *= Math.sqrt(t.BB / Math.max(o.BB, 1e-12));
    strike *= Math.sqrt(t.K / Math.max(o.K, 1e-12));
    hbp *= t.HBP / Math.max(o.HBP, 1e-12);
    const tot = ball + strike + hbp;
    if (tot > room) { const s = room / tot; ball *= s; strike *= s; hbp *= s; }
  }
  const p = { ball, strike, foul: FOUL, hbp, inPlay: 1 - FOUL - ball - strike - hbp };
  cache.set(key, p);
  return p;
}

function normalize(p: PitchProbs): PitchProbs {
  const s = p.ball + p.strike + p.foul + p.hbp + p.inPlay;
  return { ball: p.ball / s, strike: p.strike / s, foul: p.foul / s, hbp: p.hbp / s, inPlay: p.inPlay / s };
}

export function actionProbs(base: PitchProbs, action: BatterAction): PitchProbs {
  switch (action) {
    case 'normal':
      return base;
    case 'take': // no swing: only balls, called strikes and HBP remain
      return normalize({ ball: base.ball, strike: base.strike * CALLED_SHARE, foul: 0, hbp: base.hbp, inPlay: 0 });
    case 'protect': { // two-strike approach: spoil pitches, miss less, hit with less authority (see inPlayWeights)
      const called = base.strike * CALLED_SHARE;
      const swinging = base.strike - called;
      return normalize({ ball: base.ball, strike: called + swinging * 0.6, foul: base.foul * 1.6, hbp: base.hbp, inPlay: base.inPlay });
    }
    case 'bunt': // squared around: more contact, foul bunts with two strikes are strikeouts (game.ts)
      return normalize({ ball: base.ball, strike: 0.15, foul: 0.25, hbp: base.hbp, inPlay: 0.35 });
  }
}

export function samplePitch(p: PitchProbs, r: number): PitchResult {
  let c = p.ball;
  if (r < c) return 'ball';
  if (r < (c += p.strike)) return 'strike';
  if (r < (c += p.foul)) return 'foul';
  if (r < (c += p.hbp)) return 'hbp';
  return 'inPlay';
}

export function inPlayWeights(t: PaTargets, action: BatterAction): Record<InPlayKey, number> {
  const w: Record<InPlayKey, number> = { HR: t.HR, '3B': t['3B'], '2B': t['2B'], '1B': t['1B'], ROE: t.ROE, GO: t.GO, FO: t.FO, LO: t.LO };
  if (action === 'protect') { w.HR *= 0.5; w['3B'] *= 0.8; w['2B'] *= 0.8; }
  return w;
}

export function pick<K extends string>(w: Record<K, number>, r: number): K {
  const keys = Object.keys(w) as K[];
  const total = keys.reduce((a, k) => a + w[k], 0);
  let c = 0;
  for (const k of keys) { c += w[k] / total; if (r < c) return k; }
  return keys[keys.length - 1]!;
}
```

- [ ] **Step 4: Run tests**

Run: `npx vitest run tests/pitchModel.test.ts && npm run typecheck`
Expected: 5 pass; typecheck exits 0. If the solver test fails on an extreme case, print `chainOutcomes` per iteration for that case. Adjust only the update exponents (0.5 → 0.3) or the iteration cap. Never loosen the 1e-6 tolerance.

- [ ] **Step 5: Commit**

```bash
git add src/baseball/pitchModel.ts tests/pitchModel.test.ts
git commit -m "feat: count-chain pitch model solved to matchup targets" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 7: Runner movement for every play

**Files:**
- Create: `src/baseball/bases.ts`
- Test: `tests/bases.test.ts`

**Interfaces:**
- Consumes: `Bases`, `Runner`, `PaOutcome` (Task 4); `InPlayKey` (Task 6).
- Produces:
  - `type PlayKind = InPlayKey | 'BB' | 'HBP' | 'BUNT'`
  - `interface PlayCtx { bases: Bases; outs: number; batter: Runner; speed: (playerId: string) => number; ofArm: number; draw: () => number }`
  - `interface PlayResult { bases: Bases; outsAdded: number; scored: Runner[]; outcome: PaOutcome }`
  - `resolvePlay(kind: PlayKind, c: PlayCtx): PlayResult`

Randomness comes through `draw()`. The game passes `() => next(state.rng.baseball)`, and tests pass a scripted queue. If a play's outs end the inning, no runs score and the bases clear. M1 has no outs on the bases during extra-base attempts (deferred).

- [ ] **Step 1: Write the failing tests**

`tests/bases.test.ts`:
```ts
import { resolvePlay, type PlayCtx } from '../src/baseball/bases';
import type { Bases, Runner } from '../src/baseball/types';

const R = (id: string): Runner => ({ playerId: id, pitcherId: 'p', onError: false });
const ctx = (bases: Bases, outs: number, draws: number[] = []): PlayCtx => ({
  bases, outs, batter: R('bat'), speed: () => 65, ofArm: 65,
  draw: () => (draws.length ? draws.shift()! : 0.5),
});
const ids = (b: Bases) => b.map((r) => r?.playerId ?? null);

test('walk with the bases loaded forces in a run', () => {
  const r = resolvePlay('BB', ctx([R('a'), R('b'), R('c')], 0));
  expect(r.scored.map((x) => x.playerId)).toEqual(['c']);
  expect(ids(r.bases)).toEqual(['bat', 'a', 'b']);
});

test('walk with a runner on second only does not move him', () => {
  const r = resolvePlay('BB', ctx([null, R('b'), null], 0));
  expect(r.scored).toHaveLength(0);
  expect(ids(r.bases)).toEqual(['bat', 'b', null]);
});

test('single: runner on second scores or stops at third by the draw', () => {
  expect(resolvePlay('1B', ctx([null, R('b'), null], 0, [0])).scored).toHaveLength(1);
  const held = resolvePlay('1B', ctx([null, R('b'), null], 0, [0.99]));
  expect(held.scored).toHaveLength(0);
  expect(ids(held.bases)).toEqual(['bat', null, 'b']);
});

test('grand slam clears the bases', () => {
  const r = resolvePlay('HR', ctx([R('a'), R('b'), R('c')], 1));
  expect(r.scored).toHaveLength(4);
  expect(ids(r.bases)).toEqual([null, null, null]);
});

test('double play with nobody out still scores the runner from third', () => {
  const r = resolvePlay('GO', ctx([R('a'), null, R('c')], 0, [0]));
  expect(r.outcome).toBe('GDP');
  expect(r.outsAdded).toBe(2);
  expect(r.scored.map((x) => x.playerId)).toEqual(['c']);
  expect(ids(r.bases)).toEqual([null, null, null]);
});

test('an inning-ending double play scores nobody', () => {
  const r = resolvePlay('GO', ctx([R('a'), null, R('c')], 1, [0]));
  expect(r.outcome).toBe('GDP');
  expect(r.scored).toHaveLength(0);
});

test('sacrifice fly scores the runner from third with one out', () => {
  const r = resolvePlay('FO', ctx([null, null, R('c')], 1, [0]));
  expect(r.outcome).toBe('SF');
  expect(r.scored).toHaveLength(1);
});

test('a two-out fly ball scores nobody', () => {
  const r = resolvePlay('FO', ctx([null, null, R('c')], 2, [0]));
  expect(r.outcome).toBe('FO');
  expect(r.scored).toHaveLength(0);
});

test('sacrifice bunt moves the runner up and retires the batter', () => {
  const r = resolvePlay('BUNT', ctx([R('a'), null, null], 0, [0.5]));
  expect(r.outcome).toBe('SAC');
  expect(r.outsAdded).toBe(1);
  expect(ids(r.bases)).toEqual([null, 'a', null]);
});

test('reaching on error moves everyone one base', () => {
  const r = resolvePlay('ROE', ctx([R('a'), R('b'), R('c')], 0));
  expect(r.outcome).toBe('ROE');
  expect(r.scored.map((x) => x.playerId)).toEqual(['c']);
  expect(ids(r.bases)).toEqual(['bat', 'a', 'b']);
});
```

- [ ] **Step 2: Run to verify failure**

Run: `npx vitest run tests/bases.test.ts`
Expected: FAIL (cannot resolve `../src/baseball/bases`).

- [ ] **Step 3: Implement**

`src/baseball/bases.ts`:
```ts
import type { Bases, PaOutcome, Runner } from './types';
import type { InPlayKey } from './pitchModel';

export type PlayKind = InPlayKey | 'BB' | 'HBP' | 'BUNT';
export interface PlayCtx {
  bases: Bases; outs: number; batter: Runner;
  speed: (playerId: string) => number;
  ofArm: number;        // fielding team's average outfield arm
  draw: () => number;   // uniform [0, 1)
}
export interface PlayResult { bases: Bases; outsAdded: number; scored: Runner[]; outcome: PaOutcome }

const clamp = (x: number, lo = 0.02, hi = 0.98) => Math.min(hi, Math.max(lo, x));
// Extra-base chance: base rate, faster runners go more, stronger arms hold them.
const go = (c: PlayCtx, r: Runner, base: number) =>
  c.draw() < clamp(base + 0.006 * (c.speed(r.playerId) - 65) - 0.004 * (c.ofArm - 65));

function finish(c: PlayCtx, bases: Bases, scored: Runner[], outsAdded: number, outcome: PaOutcome): PlayResult {
  if (c.outs + outsAdded >= 3) return { bases: [null, null, null], outsAdded, scored: [], outcome };
  return { bases, outsAdded, scored, outcome };
}

// Batter to first; runners move only when forced.
function forced(c: PlayCtx, outcome: PaOutcome): PlayResult {
  const b: Bases = [...c.bases];
  const scored: Runner[] = [];
  if (b[0]) {
    if (b[1]) { if (b[2]) scored.push(b[2]); b[2] = b[1]; }
    b[1] = b[0];
  }
  b[0] = c.batter;
  return finish(c, b, scored, 0, outcome);
}

// Every runner moves up exactly one base; batter to `batterBase` (or out if null).
function oneBase(c: PlayCtx, batterBase: 0 | null, outsAdded: number, outcome: PaOutcome): PlayResult {
  const [r1, r2, r3] = c.bases;
  const scored: Runner[] = r3 ? [r3] : [];
  const b: Bases = [batterBase === 0 ? c.batter : null, r1, r2];
  return finish(c, b, scored, outsAdded, outcome);
}

export function resolvePlay(kind: PlayKind, c: PlayCtx): PlayResult {
  const [r1, r2, r3] = c.bases;
  const scored: Runner[] = [];
  const b: Bases = [null, null, null];
  switch (kind) {
    case 'BB': return forced(c, 'BB');
    case 'HBP': return forced(c, 'HBP');
    case 'ROE': return oneBase(c, 0, 0, 'ROE');
    case '1B': {
      if (r3) scored.push(r3);
      if (r2) { if (go(c, r2, 0.62)) scored.push(r2); else b[2] = r2; }
      if (r1) { if (!b[2] && go(c, r1, 0.28)) b[2] = r1; else b[1] = r1; }
      b[0] = c.batter;
      return finish(c, b, scored, 0, '1B');
    }
    case '2B': {
      if (r3) scored.push(r3);
      if (r2) scored.push(r2);
      if (r1) { if (go(c, r1, 0.42)) scored.push(r1); else b[2] = r1; }
      b[1] = c.batter;
      return finish(c, b, scored, 0, '2B');
    }
    case '3B': {
      for (const r of [r3, r2, r1]) if (r) scored.push(r);
      b[2] = c.batter;
      return finish(c, b, scored, 0, '3B');
    }
    case 'HR': {
      for (const r of [r3, r2, r1]) if (r) scored.push(r);
      scored.push(c.batter);
      return finish(c, b, scored, 0, 'HR');
    }
    case 'GO': {
      if (r1 && c.outs < 2) {
        if (c.draw() < clamp(0.3 - 0.005 * (c.speed(c.batter.playerId) - 65))) {
          if (r3) scored.push(r3);
          if (r2) b[2] = r2;
          return finish(c, b, scored, 2, 'GDP');
        }
        if (c.draw() < 0.4) { // fielder's choice: runner from first out at second, batter safe
          if (r3) { if (go(c, r3, 0.5)) scored.push(r3); else b[2] = r3; }
          if (r2) { if (!b[2]) b[2] = r2; else b[1] = r2; }
          b[0] = c.batter;
          return finish(c, b, scored, 1, 'FC');
        }
        if (r3) { if (go(c, r3, 0.5)) scored.push(r3); else b[2] = r3; }
        if (r2) { if (!b[2]) b[2] = r2; else b[1] = r2; }
        if (!b[1]) b[1] = r1; else b[0] = r1; // ponytail: rare traffic jam keeps the runner on first
        return finish(c, b, scored, 1, 'GO');
      }
      if (r3) { if (go(c, r3, 0.5)) scored.push(r3); else b[2] = r3; }
      if (r2) { if (!b[2] && go(c, r2, 0.55)) b[2] = r2; else b[1] = r2; }
      if (r1) { if (!b[1]) b[1] = r1; else b[0] = r1; }
      return finish(c, b, scored, 1, 'GO');
    }
    case 'FO': {
      let outcome: PaOutcome = 'FO';
      if (r3) { if (c.outs < 2 && go(c, r3, 0.55)) { scored.push(r3); outcome = 'SF'; } else b[2] = r3; }
      if (r2) { if (!b[2] && c.outs < 2 && go(c, r2, 0.2)) b[2] = r2; else b[1] = r2; }
      b[0] = r1;
      return finish(c, b, scored, 1, outcome);
    }
    case 'LO':
      return finish(c, [...c.bases], scored, 1, 'LO');
    case 'BUNT': {
      const r = c.draw();
      const hit = clamp(0.06 + 0.006 * (c.speed(c.batter.playerId) - 65), 0.01, 0.3);
      if (r < hit) return oneBase(c, 0, 0, 'BUNT_HIT');
      if (!r1 && !r2 && !r3) return finish(c, b, scored, 1, 'GO');
      if (r < hit + 0.75) return oneBase(c, null, 1, 'SAC');
      // Failed bunt: lead forced runner out, batter safe at first.
      if (r1 && r2) return finish(c, [c.batter, r1, r3], scored, 1, 'FC');
      if (r1) return finish(c, [c.batter, r2, r3], scored, 1, 'FC');
      return finish(c, [null, r2, r3], scored, 1, 'GO');
    }
  }
}
```

- [ ] **Step 4: Run tests**

Run: `npx vitest run tests/bases.test.ts && npm run typecheck`
Expected: 10 pass; typecheck exits 0.

- [ ] **Step 5: Commit**

```bash
git add src/baseball/bases.ts tests/bases.test.ts
git commit -m "feat: runner movement for hits, outs, walks, errors and bunts" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 8: The game state machine

**Files:**
- Create: `src/baseball/game.ts`
- Test: `tests/game.test.ts`

**Interfaces:**
- Consumes: everything from Tasks 2–7.
- Produces:
  - `INNINGS` (9)
  - `newGame(away: Team, home: Team, year: number, seed: string): GameState`
  - `legalActions(s: GameState): Action[]`
  - `applyAction(s: GameState, a: Action): void` (mutates; throws on an illegal action)
  - Read-only helpers: `battingSide(s)`, `fieldingSide(s)`, `player(s, side, id)`, `currentBatter(s)`, `currentPitcher(s)`, `pitchLimit(p)`, `fatigue(s, side)`, `targetsFor(s)`, `availablePitchers(s, side)`, `availableBench(s, side)`
  - Test seam: `setTestDraws(q: number[] | null)`. While set, scripted draws replace the baseball stream.

The rules this enforces:
- **Count:** four balls is a walk and three strikes is a strikeout. A foul adds a strike only below two strikes, and a foul bunt with two strikes is a strikeout.
- **End of game:**
  - The bottom of the 9th or later ends the instant the home team leads (a walk-off). Only the winning run counts, except on a home run.
  - There is no bottom of the 9th if the home team leads.
  - A tie after nine goes to extra innings.
- **Timing of moves:** pitching changes, pinch hitters and intentional walks happen only at the start of a plate appearance. Steals can happen between pitches.
- **Substitutions:**
  - A removed player cannot re-enter.
  - Pinch-hitting for the pitcher forces a pitching change before that team pitches again.
- **Caught stealing for the third out:** the batter leads off the next inning.
- **Runs:** each run is charged to the pitcher who put that runner on base. A runner who reached on an error scores an unearned run.

- [ ] **Step 1: Write the failing tests**

`tests/game.test.ts`:
```ts
import { newGame, applyAction, legalActions, setTestDraws, targetsFor } from '../src/baseball/game';
import { solvePitchProbs, actionProbs, inPlayWeights } from '../src/baseball/pitchModel';
import { makeTeam } from '../src/baseball/fixtures';
import type { GameState, Runner } from '../src/baseball/types';

afterEach(() => setTestDraws(null));
const fresh = () => newGame(makeTeam('aw', 'Away'), makeTeam('hm', 'Home'), 1918, 'rules');
const R = (id: string): Runner => ({ playerId: id, pitcherId: 'aw-p1', onError: false });
const probs = (s: GameState) => solvePitchProbs(targetsFor(s));
const BALL = 0;
const IN_PLAY = 0.9999999;
const strike = (s: GameState) => probs(s).ball + 1e-9;
const foul = (s: GameState) => { const p = probs(s); return p.ball + p.strike + 1e-9; };
const pitch = (s: GameState) => applyAction(s, { kind: 'pitch', batter: 'normal' });
function bottomOf(s: GameState, inning: number, away: number, home: number) {
  s.inning = inning; s.half = 'bottom'; s.score = { away, home };
  s.lineScore = { away: [away, ...Array(inning - 1).fill(0)], home: [home, ...Array(inning - 1).fill(0)] };
}
const pas = (s: GameState) => s.events.filter((e) => e.t === 'pa');

test('four balls is a walk', () => {
  const s = fresh();
  setTestDraws([BALL, BALL, BALL, BALL]);
  for (let i = 0; i < 4; i++) pitch(s);
  expect(pas(s)).toHaveLength(1);
  expect(pas(s)[0]).toMatchObject({ outcome: 'BB' });
  expect(s.bases[0]?.playerId).toBe('aw-b1');
  expect(s.sides.away.battingIndex).toBe(1);
  expect([s.balls, s.strikes]).toEqual([0, 0]);
});

test('three strikes is a strikeout', () => {
  const s = fresh();
  const x = strike(s);
  setTestDraws([x, x, x]);
  for (let i = 0; i < 3; i++) pitch(s);
  expect(pas(s)[0]).toMatchObject({ outcome: 'K' });
  expect(s.outs).toBe(1);
});

test('a two-strike foul keeps the count', () => {
  const s = fresh();
  s.strikes = 2;
  setTestDraws([foul(s)]);
  pitch(s);
  expect(s.strikes).toBe(2);
  expect(pas(s)).toHaveLength(0);
});

test('a two-strike foul bunt is a strikeout', () => {
  const s = fresh();
  s.strikes = 2;
  const b = actionProbs(probs(s), 'bunt');
  setTestDraws([b.ball + b.strike + 1e-9]);
  applyAction(s, { kind: 'pitch', batter: 'bunt' });
  expect(pas(s)[0]).toMatchObject({ outcome: 'K' });
});

test('walk-off walk counts only the winning run', () => {
  const s = fresh();
  bottomOf(s, 9, 2, 2);
  s.bases = [R('hm-b5'), R('hm-b6'), R('hm-b7')];
  s.balls = 3;
  setTestDraws([BALL]);
  pitch(s);
  expect(s.final).toBe(true);
  expect(s.score).toEqual({ away: 2, home: 3 });
});

test('walk-off grand slam counts every run', () => {
  const s = fresh();
  bottomOf(s, 9, 5, 2);
  s.bases = [R('hm-b5'), R('hm-b6'), R('hm-b7')];
  setTestDraws([IN_PLAY, 0]); // in play, then the first in-play bucket (HR)
  pitch(s);
  expect(s.final).toBe(true);
  expect(s.score).toEqual({ away: 5, home: 6 });
});

test('a walk-off double counts only the winning run', () => {
  const s = fresh();
  bottomOf(s, 10, 1, 1);
  s.bases = [null, R('hm-b5'), R('hm-b6')];
  const w = inPlayWeights(targetsFor(s), 'normal');
  const total = Object.values(w).reduce((a, b) => a + b, 0);
  setTestDraws([IN_PLAY, (w.HR + w['3B'] + w['2B'] / 2) / total]);
  pitch(s);
  expect(s.final).toBe(true);
  expect(s.score).toEqual({ away: 1, home: 2 });
});

test('no bottom of the ninth when the home team leads', () => {
  const s = fresh();
  s.inning = 9; s.half = 'top'; s.outs = 2; s.strikes = 2; s.score = { away: 1, home: 3 };
  s.lineScore = { away: [1, 0, 0, 0, 0, 0, 0, 0, 0], home: [3, 0, 0, 0, 0, 0, 0, 0] };
  setTestDraws([strike(s)]);
  pitch(s);
  expect(s.final).toBe(true);
  expect(s.lineScore.home).toHaveLength(8);
});

test('a tie after nine goes to extra innings', () => {
  const s = fresh();
  bottomOf(s, 9, 1, 1);
  s.outs = 2; s.strikes = 2;
  setTestDraws([strike(s)]);
  pitch(s);
  expect(s.final).toBe(false);
  expect([s.inning, s.half]).toEqual([10, 'top']);
});

test('pinch-hitting for the pitcher forces a change, and removed players stay out', () => {
  const s = fresh();
  s.sides.away.battingIndex = 8;
  applyAction(s, { kind: 'pinchHit', inId: 'aw-bench1' });
  expect(s.sides.away.needsPitcher).toBe(true);
  s.outs = 2; s.strikes = 2;
  setTestDraws([strike(s)]);
  pitch(s);
  expect(s.half).toBe('bottom');
  const acts = legalActions(s);
  expect(acts.length).toBeGreaterThan(0);
  expect(acts.every((a) => a.kind === 'changePitcher')).toBe(true);
  applyAction(s, { kind: 'changePitcher', inId: 'aw-p2' });
  expect(s.sides.away.lineup[8]!.playerId).toBe('aw-p2');
  expect(s.sides.away.removed).toEqual(expect.arrayContaining(['aw-p1', 'aw-bench1']));
  expect(legalActions(s).some((a) => a.kind === 'changePitcher' && a.inId === 'aw-p1')).toBe(false);
});

test('caught stealing for the third out keeps the batter for next inning', () => {
  const s = fresh();
  s.outs = 2;
  s.bases = [R('aw-b1'), null, null];
  s.sides.away.battingIndex = 1;
  setTestDraws([0.9999999]);
  applyAction(s, { kind: 'steal', from: 1 });
  expect(s.half).toBe('bottom');
  expect(s.sides.away.battingIndex).toBe(1);
});

test('illegal actions throw', () => {
  const s = fresh();
  expect(() => applyAction(s, { kind: 'steal', from: 1 })).toThrow(/illegal/);
});

test('a run is unearned when the runner reached on an error', () => {
  const s = fresh();
  s.bases = [null, null, { playerId: 'aw-b3', pitcherId: 'hm-p1', onError: true }];
  s.balls = 3;
  s.bases[0] = R('aw-b1'); s.bases[1] = R('aw-b2');
  setTestDraws([BALL]);
  pitch(s);
  const run = s.events.find((e) => e.t === 'run');
  expect(run).toMatchObject({ runnerId: 'aw-b3', earned: false, pitcherId: 'hm-p1' });
});
```

- [ ] **Step 2: Run to verify failure**

Run: `npx vitest run tests/game.test.ts`
Expected: FAIL (cannot resolve `../src/baseball/game`).

- [ ] **Step 3: Implement**

`src/baseball/game.ts`:
```ts
import type { Action, BatterAction, GameState, Player, Runner, Side, SideState, Team } from './types';
import { makeStreams, next } from '../core/rng';
import { getEra } from './eras';
import { paTargets, type PaTargets } from './matchup';
import { actionProbs, inPlayWeights, pick, samplePitch, solvePitchProbs } from './pitchModel';
import { resolvePlay, type PlayKind, type PlayResult } from './bases';

export const INNINGS = 9;
const BATTER_ACTIONS: BatterAction[] = ['normal', 'take', 'protect', 'bunt'];
const FIELDERS = ['C', '1B', '2B', '3B', 'SS', 'LF', 'CF', 'RF'];

// Test seam: while set, scripted draws replace the baseball stream. ponytail: module-level, tests only.
let scripted: number[] | null = null;
export function setTestDraws(q: number[] | null): void { scripted = q; }
function draw(s: GameState): number {
  return scripted && scripted.length ? scripted.shift()! : next(s.rng.baseball);
}

export function newGame(away: Team, home: Team, year: number, seed: string): GameState {
  getEra(year); // throws outside 1918–1930
  const side = (t: Team): SideState => ({
    lineup: t.lineup.map((x) => ({ ...x })), battingIndex: 0, pitcherId: t.startingPitcherId,
    pitchCount: 0, runsAllowed: 0, removed: [], needsPitcher: false,
  });
  return {
    version: 1, year,
    teams: { away: structuredClone(away), home: structuredClone(home) },
    sides: { away: side(away), home: side(home) },
    inning: 1, half: 'top', outs: 0, balls: 0, strikes: 0,
    bases: [null, null, null],
    score: { away: 0, home: 0 },
    lineScore: { away: [0], home: [] },
    rng: makeStreams(seed),
    events: [{ t: 'half', inning: 1, half: 'top' }],
    final: false,
  };
}

export const battingSide = (s: GameState): Side => (s.half === 'top' ? 'away' : 'home');
export const fieldingSide = (s: GameState): Side => (s.half === 'top' ? 'home' : 'away');

export function player(s: GameState, side: Side, id: string): Player {
  const p = s.teams[side].players.find((x) => x.id === id);
  if (!p) throw new Error(`no player ${id} on ${side}`);
  return p;
}
export function currentBatter(s: GameState): Player {
  const side = battingSide(s);
  const st = s.sides[side];
  return player(s, side, st.lineup[st.battingIndex]!.playerId);
}
export function currentPitcher(s: GameState): Player {
  const side = fieldingSide(s);
  return player(s, side, s.sides[side].pitcherId);
}
export function pitchLimit(p: Player): number {
  return 70 + 0.8 * (p.pitching?.stamina ?? 0);
}
export function fatigue(s: GameState, side: Side): number {
  const st = s.sides[side];
  const lim = pitchLimit(player(s, side, st.pitcherId));
  return Math.min(1, Math.max(0, (st.pitchCount - 0.8 * lim) / (0.4 * lim)));
}
function avgRating(s: GameState, side: Side, positions: string[], key: 'fielding' | 'arm'): number {
  const slots = s.sides[side].lineup.filter((x) => positions.includes(x.pos));
  return slots.reduce((a, x) => a + player(s, side, x.playerId).batting[key], 0) / Math.max(1, slots.length);
}
export function targetsFor(s: GameState): PaTargets {
  const b = currentBatter(s);
  const p = currentPitcher(s);
  const fld = fieldingSide(s);
  return paTargets({
    batter: b.batting, bats: b.bats, pitcher: p.pitching!, throws: p.throws,
    fatigue: fatigue(s, fld), defenseFielding: avgRating(s, fld, FIELDERS, 'fielding'), era: getEra(s.year),
  });
}

const used = (st: SideState) => new Set([...st.lineup.map((x) => x.playerId), ...st.removed, st.pitcherId]);
export function availablePitchers(s: GameState, side: Side): Player[] {
  const u = used(s.sides[side]);
  return s.teams[side].players.filter((p) => p.pitching && !u.has(p.id));
}
export function availableBench(s: GameState, side: Side): Player[] {
  const u = used(s.sides[side]);
  return s.teams[side].players.filter((p) => !p.pitching && !u.has(p.id));
}

export function legalActions(s: GameState): Action[] {
  if (s.final) return [];
  const bat = battingSide(s);
  const fld = fieldingSide(s);
  const acts: Action[] = [];
  const paStart = s.balls === 0 && s.strikes === 0;
  const mustChange = s.sides[fld].needsPitcher;
  if (paStart || mustChange) for (const p of availablePitchers(s, fld)) acts.push({ kind: 'changePitcher', inId: p.id });
  if (mustChange) return acts;
  if (paStart) {
    acts.push({ kind: 'ibb' });
    const slot = s.sides[bat].lineup[s.sides[bat].battingIndex]!;
    // Pinch-hitting for the pitcher needs a pitcher left to replace him.
    if (slot.pos !== 'P' || availablePitchers(s, bat).length > 0) {
      for (const p of availableBench(s, bat)) acts.push({ kind: 'pinchHit', inId: p.id });
    }
  }
  if (s.bases[0] && !s.bases[1]) acts.push({ kind: 'steal', from: 1 });
  if (s.bases[1] && !s.bases[2]) acts.push({ kind: 'steal', from: 2 });
  for (const b of BATTER_ACTIONS) acts.push({ kind: 'pitch', batter: b });
  return acts;
}

function same(a: Action, b: Action): boolean {
  if (a.kind !== b.kind) return false;
  switch (a.kind) {
    case 'pitch': return a.batter === (b as typeof a).batter;
    case 'steal': return a.from === (b as typeof a).from;
    case 'pinchHit':
    case 'changePitcher': return a.inId === (b as typeof a).inId;
    case 'ibb': return true;
  }
}

export function applyAction(s: GameState, a: Action): void {
  if (!legalActions(s).some((x) => same(x, a))) throw new Error(`illegal action ${JSON.stringify(a)}`);
  switch (a.kind) {
    case 'changePitcher': return changePitcher(s, a.inId);
    case 'pinchHit': return pinchHit(s, a.inId);
    case 'ibb': return endPa(s, { ...play(s, 'BB'), outcome: 'IBB' });
    case 'steal': return steal(s, a.from);
    case 'pitch': return pitch(s, a.batter);
  }
}

function play(s: GameState, kind: PlayKind): PlayResult {
  const bat = battingSide(s);
  const fld = fieldingSide(s);
  const batter: Runner = { playerId: currentBatter(s).id, pitcherId: s.sides[fld].pitcherId, onError: kind === 'ROE' };
  return resolvePlay(kind, {
    bases: s.bases, outs: s.outs, batter,
    speed: (id) => player(s, bat, id).batting.speed,
    ofArm: avgRating(s, fld, ['LF', 'CF', 'RF'], 'arm'),
    draw: () => draw(s),
  });
}

function pitch(s: GameState, action: BatterAction): void {
  const fld = fieldingSide(s);
  const t = targetsFor(s);
  const result = samplePitch(actionProbs(solvePitchProbs(t), action), draw(s));
  s.sides[fld].pitchCount++;
  if (result === 'ball') s.balls++;
  else if (result === 'strike') s.strikes++;
  else if (result === 'foul') {
    if (action === 'bunt' && s.strikes === 2) s.strikes = 3;
    else if (s.strikes < 2) s.strikes++;
  }
  s.events.push({ t: 'pitch', result, batterId: currentBatter(s).id, pitcherId: s.sides[fld].pitcherId, balls: s.balls, strikes: s.strikes });
  if (s.balls === 4) return endPa(s, play(s, 'BB'));
  if (s.strikes === 3) return endPa(s, { bases: s.bases, outsAdded: 1, scored: [], outcome: 'K' });
  if (result === 'hbp') return endPa(s, play(s, 'HBP'));
  if (result === 'inPlay') {
    const kind: PlayKind = action === 'bunt' ? 'BUNT' : pick(inPlayWeights(t, action), draw(s));
    return endPa(s, play(s, kind));
  }
}

function steal(s: GameState, from: 1 | 2): void {
  const bat = battingSide(s);
  const fld = fieldingSide(s);
  const runner = s.bases[from - 1]!;
  const speed = player(s, bat, runner.playerId).batting.speed;
  const catcherArm = avgRating(s, fld, ['C'], 'arm');
  const p = Math.min(0.95, Math.max(0.05, 1 - getEra(s.year).csRate + 0.008 * (speed - 65) - 0.005 * (catcherArm - 65)));
  const success = draw(s) < p;
  s.events.push({ t: 'steal', runnerId: runner.playerId, pitcherId: s.sides[fld].pitcherId, from, success, side: bat });
  s.bases[from - 1] = null;
  if (success) s.bases[from] = runner;
  else if (++s.outs >= 3) endHalf(s);
}

function endPa(s: GameState, r: PlayResult): void {
  const bat = battingSide(s);
  const fld = fieldingSide(s);
  let scored = r.scored;
  if (s.half === 'bottom' && s.inning >= INNINGS && r.outcome !== 'HR') {
    const need = s.score.away - s.score.home + 1; // runs that win it
    if (need > 0 && scored.length > need) scored = scored.slice(0, need);
  }
  s.events.push({
    t: 'pa', outcome: r.outcome, batterId: currentBatter(s).id, pitcherId: s.sides[fld].pitcherId,
    outs: r.outsAdded, rbi: r.outcome === 'GDP' || r.outcome === 'ROE' ? 0 : scored.length, side: bat,
  });
  for (const run of scored) scoreRun(s, run);
  s.bases = r.bases;
  s.outs += r.outsAdded;
  s.balls = 0;
  s.strikes = 0;
  const st = s.sides[bat];
  st.battingIndex = (st.battingIndex + 1) % 9;
  if (s.half === 'bottom' && s.inning >= INNINGS && s.score.home > s.score.away) return finalize(s);
  if (s.outs >= 3) endHalf(s);
}

function scoreRun(s: GameState, r: Runner): void {
  const bat = battingSide(s);
  const fld = fieldingSide(s);
  s.score[bat]++;
  const line = s.lineScore[bat];
  line[line.length - 1] = line[line.length - 1]! + 1;
  if (r.pitcherId === s.sides[fld].pitcherId) s.sides[fld].runsAllowed++;
  s.events.push({ t: 'run', runnerId: r.playerId, pitcherId: r.pitcherId, earned: !r.onError, side: bat });
}

function endHalf(s: GameState): void {
  if (s.half === 'top') {
    if (s.inning >= INNINGS && s.score.home > s.score.away) return finalize(s);
    s.half = 'bottom';
    s.lineScore.home.push(0);
  } else {
    if (s.inning >= INNINGS && s.score.home !== s.score.away) return finalize(s);
    s.inning++;
    s.half = 'top';
    s.lineScore.away.push(0);
  }
  s.outs = 0;
  s.balls = 0;
  s.strikes = 0;
  s.bases = [null, null, null];
  s.events.push({ t: 'half', inning: s.inning, half: s.half });
}

function finalize(s: GameState): void {
  s.final = true;
  s.events.push({ t: 'final', away: s.score.away, home: s.score.home, innings: s.inning });
}

function changePitcher(s: GameState, inId: string): void {
  const fld = fieldingSide(s);
  const st = s.sides[fld];
  const slot = st.lineup.find((x) => x.pos === 'P')!;
  const outId = st.pitcherId;
  for (const id of [outId, slot.playerId]) if (!st.removed.includes(id)) st.removed.push(id);
  slot.playerId = inId;
  st.pitcherId = inId;
  st.pitchCount = 0;
  st.runsAllowed = 0;
  st.needsPitcher = false;
  s.events.push({ t: 'sub', kind: 'P', inId, outId, side: fld });
}

// ponytail: the pinch hitter stays in at the replaced position; no double switches in M1.
function pinchHit(s: GameState, inId: string): void {
  const bat = battingSide(s);
  const st = s.sides[bat];
  const slot = st.lineup[st.battingIndex]!;
  const outId = slot.playerId;
  st.removed.push(outId);
  slot.playerId = inId;
  if (slot.pos === 'P') st.needsPitcher = true;
  s.events.push({ t: 'sub', kind: 'PH', inId, outId, side: bat });
}
```

- [ ] **Step 4: Run tests**

Run: `npx vitest run tests/game.test.ts && npm run typecheck`
Expected: 13 pass; typecheck exits 0.

- [ ] **Step 5: Commit**

```bash
git add src/baseball/game.ts tests/game.test.ts
git commit -m "feat: game state machine with walk-offs, extras, steals and substitutions" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 9: AI manager, full-game sim, determinism and mode equivalence

**Files:**
- Create: `src/baseball/manager.ts`, `src/baseball/sim.ts`
- Test: `tests/sim.test.ts`

**Interfaces:**
- Consumes: `legalActions`, `applyAction`, `newGame` and the read-only helpers (Task 8); `next` (Task 2).
- Produces:
  - `KNOBS` (mutable tuning object: `stealAttempt`, `sacBunt`, `pullFatigue`, `pullRuns`)
  - `aiAction(s: GameState): Action`, which draws only from `s.rng.manager`
  - `simulateGame(away, home, year, seed): { state: GameState; actions: Action[] }`
  - `replay(away, home, year, seed, actions): GameState`

The AI manages in a period style:
- It leaves a starter in until he tires or gets hit hard.
- It walks dangerous hitters intentionally, but only late in close games with first base open.
- It pinch-hits for the pitcher late when trailing.
- It sends fast runners.
- It bunts runners over with nobody out.

"Mode equivalence" means that replaying the exact actions the AI chose gives an identical game. That's the property that lets a player take over any game, or any pitch, without changing the odds.

- [ ] **Step 1: Write the failing tests**

`tests/sim.test.ts`:
```ts
import { simulateGame, replay } from '../src/baseball/sim';
import { makeTeam } from '../src/baseball/fixtures';

const away = makeTeam('aw', 'Away');
const home = makeTeam('hm', 'Home');

test('every game finishes with a winner', () => {
  for (let i = 0; i < 200; i++) {
    const { state } = simulateGame(away, home, 1918, `finish-${i}`);
    expect(state.final).toBe(true);
    expect(state.score.away).not.toBe(state.score.home);
    expect(state.events.at(-1)).toMatchObject({ t: 'final' });
    expect(state.inning).toBeGreaterThanOrEqual(9);
  }
});

test('the same seed gives the same game', () => {
  const a = simulateGame(away, home, 1925, 'same').state;
  const b = simulateGame(away, home, 1925, 'same').state;
  expect(b.events).toEqual(a.events);
  expect(b.score).toEqual(a.score);
});

test('different seeds give different games', () => {
  const scores = new Set<string>();
  for (let i = 0; i < 20; i++) {
    const { state } = simulateGame(away, home, 1925, `diff-${i}`);
    scores.add(`${state.score.away}-${state.score.home}-${state.events.length}`);
  }
  expect(scores.size).toBeGreaterThan(10);
});

test('replaying the same decisions gives the identical game (mode equivalence)', () => {
  for (let i = 0; i < 25; i++) {
    const { state, actions } = simulateGame(away, home, 1920, `equiv-${i}`);
    const again = replay(away, home, 1920, `equiv-${i}`, actions);
    expect(again.events).toEqual(state.events);
    expect(again.rng.baseball).toEqual(state.rng.baseball);
  }
});

test('the AI uses the whole playbook across many games', () => {
  const kinds = new Set<string>();
  for (let i = 0; i < 300; i++) {
    for (const a of simulateGame(away, home, 1918, `book-${i}`).actions) {
      kinds.add(a.kind === 'pitch' ? `pitch:${a.batter}` : a.kind);
    }
  }
  for (const k of ['pitch:normal', 'pitch:bunt', 'steal', 'changePitcher', 'pinchHit']) expect(kinds).toContain(k);
});
```

- [ ] **Step 2: Run to verify failure**

Run: `npx vitest run tests/sim.test.ts`
Expected: FAIL (cannot resolve `../src/baseball/sim`).

- [ ] **Step 3: Implement the manager**

`src/baseball/manager.ts`:
```ts
import type { Action, GameState, Player } from './types';
import { next } from '../core/rng';
import { availablePitchers, battingSide, currentBatter, fatigue, fieldingSide, legalActions, player } from './game';

// Tuned in Task 12 so simmed seasons match each era's steal and sacrifice rates.
export const KNOBS = { stealAttempt: 0.08, sacBunt: 0.6, pullFatigue: 0.65, pullRuns: 7 };

const pitchScore = (p: Player) => (p.pitching ? p.pitching.control + p.pitching.stuff + p.pitching.contactSuppression : 0);
const batScore = (p: Player) => p.batting.contact + p.batting.power + p.batting.eye;
const best = (ps: Player[], f: (p: Player) => number) => ps.reduce((a, b) => (f(b) > f(a) ? b : a));

export function aiAction(s: GameState): Action {
  const acts = legalActions(s);
  if (!acts.length) throw new Error('no legal actions: the game is over');
  const bat = battingSide(s);
  const fld = fieldingSide(s);
  const paStart = s.balls === 0 && s.strikes === 0;
  const roll = () => next(s.rng.manager);

  // Fielding team: change pitchers when forced, tired, or getting hit hard.
  const relievers = availablePitchers(s, fld);
  const st = s.sides[fld];
  if (relievers.length && (st.needsPitcher || (paStart && (fatigue(s, fld) > KNOBS.pullFatigue || st.runsAllowed >= KNOBS.pullRuns)))) {
    return { kind: 'changePitcher', inId: best(relievers, pitchScore).id };
  }

  // Intentional walk: late, close, first base open, runner in scoring position, a dangerous bat.
  const close = Math.abs(s.score.away - s.score.home) <= 1;
  if (paStart && s.inning >= 8 && close && !s.bases[0] && (s.bases[1] || s.bases[2]) && currentBatter(s).batting.power >= 80) {
    return { kind: 'ibb' };
  }

  // Batting team: pinch-hit for the pitcher late when trailing.
  const slot = s.sides[bat].lineup[s.sides[bat].battingIndex]!;
  if (paStart && slot.pos === 'P' && s.inning >= 7 && s.score[bat] < s.score[fld]) {
    const ids = new Set(acts.flatMap((a) => (a.kind === 'pinchHit' ? [a.inId] : [])));
    const bench = s.teams[bat].players.filter((p) => ids.has(p.id));
    if (bench.length) return { kind: 'pinchHit', inId: best(bench, batScore).id };
  }

  // Steal: at the first pitch, fewer than two outs, a runner with some speed.
  if (paStart && s.outs < 2) {
    for (const from of [1, 2] as const) {
      if (!acts.some((a) => a.kind === 'steal' && a.from === from)) continue;
      const runner = s.bases[from - 1]!;
      if (player(s, bat, runner.playerId).batting.speed >= 60 && roll() < KNOBS.stealAttempt) return { kind: 'steal', from };
    }
  }

  // Sacrifice bunt: nobody out, runner on first or second, third open, pitcher or weak hitter.
  if (s.outs === 0 && (s.bases[0] || s.bases[1]) && !s.bases[2] && s.strikes < 2
      && (slot.pos === 'P' || currentBatter(s).batting.contact < 55) && roll() < KNOBS.sacBunt) {
    return { kind: 'pitch', batter: 'bunt' };
  }

  return { kind: 'pitch', batter: 'normal' };
}
```

- [ ] **Step 4: Implement the sim**

`src/baseball/sim.ts`:
```ts
import type { Action, GameState, Team } from './types';
import { applyAction, newGame } from './game';
import { aiAction } from './manager';

const MAX_ACTIONS = 20_000; // a 30-inning game is ~1,000 actions; this only catches engine bugs

export function simulateGame(away: Team, home: Team, year: number, seed: string): { state: GameState; actions: Action[] } {
  const state = newGame(away, home, year, seed);
  const actions: Action[] = [];
  while (!state.final) {
    if (actions.length >= MAX_ACTIONS) throw new Error(`game ${seed} did not finish`);
    const a = aiAction(state);
    actions.push(a);
    applyAction(state, a);
  }
  return { state, actions };
}

export function replay(away: Team, home: Team, year: number, seed: string, actions: Action[]): GameState {
  const state = newGame(away, home, year, seed);
  for (const a of actions) applyAction(state, a);
  return state;
}
```

- [ ] **Step 5: Run tests**

Run: `npx vitest run tests/sim.test.ts && npm run typecheck`
Expected: 5 pass; typecheck exits 0. If the playbook test is missing `pinchHit`, that's because the fixture teams are evenly matched, so trailing late can be rare in 300 games. Check `steal` and `pitch:bunt` frequencies with a quick count before touching the knobs; Task 12 tunes them against real data.

- [ ] **Step 6: Commit**

```bash
git add src/baseball/manager.ts src/baseball/sim.ts tests/sim.test.ts
git commit -m "feat: period-style AI manager, full-game sim, replay equivalence" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 10: Box score from the event stream

**Files:**
- Create: `src/baseball/boxScore.ts`
- Test: `tests/boxScore.test.ts`

**Interfaces:**
- Consumes: `GameState`, `GameEvent`, `PaOutcome`, `Side` (Task 4); `simulateGame` (Task 9).
- Produces:
  - `interface BatLine { playerId; pa; ab; r; h; d; t; hr; rbi; bb; k; hbp; sh; sf; sb; cs }` (ids are strings, counts numbers)
  - `interface PitchLine { playerId; outs; bf; h; r; er; bb; k; hr; pitches }`
  - `interface BoxScore { batting: Record<Side, BatLine[]>; pitching: Record<Side, PitchLine[]>; runs: Record<Side, number>; hits: Record<Side, number>; errors: Record<Side, number> }`
  - `boxScore(s: GameState): BoxScore`

Everything is folded from `s.events`. That makes the event stream the single source the newspaper and league leaders will use in M2. Errors are charged to the fielding side for each `ROE`. Earned runs are runs where the runner did not reach on an error.

- [ ] **Step 1: Write the failing tests**

`tests/boxScore.test.ts`:
```ts
import { boxScore } from '../src/baseball/boxScore';
import { simulateGame } from '../src/baseball/sim';
import { makeTeam } from '../src/baseball/fixtures';
import type { Side } from '../src/baseball/types';

const away = makeTeam('aw', 'Away');
const home = makeTeam('hm', 'Home');
const sides: Side[] = ['away', 'home'];
const other = (x: Side): Side => (x === 'away' ? 'home' : 'away');
const sum = <T>(xs: T[], f: (x: T) => number) => xs.reduce((a, x) => a + f(x), 0);

test('runs are conserved across score, line score, batting and pitching', () => {
  for (let i = 0; i < 150; i++) {
    const { state } = simulateGame(away, home, 1922, `box-${i}`);
    const box = boxScore(state);
    for (const x of sides) {
      expect(box.runs[x]).toBe(state.score[x]);
      expect(sum(state.lineScore[x], (n) => n)).toBe(state.score[x]);
      expect(sum(box.batting[x], (b) => b.r)).toBe(state.score[x]);
      expect(sum(box.pitching[other(x)], (p) => p.r)).toBe(state.score[x]);
    }
  }
});

test('plate appearances add up and outs match innings', () => {
  for (let i = 0; i < 100; i++) {
    const { state } = simulateGame(away, home, 1927, `pa-${i}`);
    const box = boxScore(state);
    for (const x of sides) {
      for (const b of box.batting[x]) {
        expect(b.pa).toBe(b.ab + b.bb + b.hbp + b.sh + b.sf);
        expect(b.h).toBeGreaterThanOrEqual(b.d + b.t + b.hr);
      }
      // The fielding side records 3 outs per completed half; a walk-off half ends early.
      const outs = sum(box.pitching[x], (p) => p.outs);
      expect(outs).toBeGreaterThanOrEqual(3 * (state.inning - 1));
      expect(outs).toBeLessThanOrEqual(3 * state.inning);
    }
  }
});

test('every player who appeared has a line, and hits match the team total', () => {
  const { state } = simulateGame(away, home, 1930, 'lines');
  const box = boxScore(state);
  for (const x of sides) {
    expect(box.batting[x].length).toBeGreaterThanOrEqual(9);
    expect(sum(box.batting[x], (b) => b.h)).toBe(box.hits[x]);
    expect(box.pitching[x].length).toBeGreaterThanOrEqual(1);
    for (const p of box.pitching[x]) expect(p.er).toBeLessThanOrEqual(p.r);
  }
});
```

- [ ] **Step 2: Run to verify failure**

Run: `npx vitest run tests/boxScore.test.ts`
Expected: FAIL (cannot resolve `../src/baseball/boxScore`).

- [ ] **Step 3: Implement**

`src/baseball/boxScore.ts`:
```ts
import type { GameState, PaOutcome, Side } from './types';

export interface BatLine { playerId: string; pa: number; ab: number; r: number; h: number; d: number; t: number; hr: number; rbi: number; bb: number; k: number; hbp: number; sh: number; sf: number; sb: number; cs: number }
export interface PitchLine { playerId: string; outs: number; bf: number; h: number; r: number; er: number; bb: number; k: number; hr: number; pitches: number }
export interface BoxScore {
  batting: Record<Side, BatLine[]>;
  pitching: Record<Side, PitchLine[]>;
  runs: Record<Side, number>;
  hits: Record<Side, number>;
  errors: Record<Side, number>;
}

const HITS: PaOutcome[] = ['1B', '2B', '3B', 'HR', 'BUNT_HIT'];
const NOT_AB: PaOutcome[] = ['BB', 'IBB', 'HBP', 'SAC', 'SF'];
const other = (x: Side): Side => (x === 'away' ? 'home' : 'away');

export function boxScore(s: GameState): BoxScore {
  const bat: Record<Side, Map<string, BatLine>> = { away: new Map(), home: new Map() };
  const pit: Record<Side, Map<string, PitchLine>> = { away: new Map(), home: new Map() };
  const errors: Record<Side, number> = { away: 0, home: 0 };
  const b = (side: Side, id: string): BatLine => {
    let l = bat[side].get(id);
    if (!l) bat[side].set(id, (l = { playerId: id, pa: 0, ab: 0, r: 0, h: 0, d: 0, t: 0, hr: 0, rbi: 0, bb: 0, k: 0, hbp: 0, sh: 0, sf: 0, sb: 0, cs: 0 }));
    return l;
  };
  const p = (side: Side, id: string): PitchLine => {
    let l = pit[side].get(id);
    if (!l) pit[side].set(id, (l = { playerId: id, outs: 0, bf: 0, h: 0, r: 0, er: 0, bb: 0, k: 0, hr: 0, pitches: 0 }));
    return l;
  };
  // Starting lineups appear even if a player never batted (e.g. lifted early).
  for (const side of ['away', 'home'] as Side[]) {
    for (const slot of s.teams[side].lineup) b(side, slot.playerId);
    p(side, s.teams[side].startingPitcherId);
  }

  for (const e of s.events) {
    if (e.t === 'pitch') {
      const fld = s.teams.away.players.some((x) => x.id === e.pitcherId) ? 'away' : 'home';
      p(fld, e.pitcherId).pitches++;
    } else if (e.t === 'pa') {
      const fld = other(e.side);
      const bl = b(e.side, e.batterId);
      const pl = p(fld, e.pitcherId);
      bl.pa++; pl.bf++; pl.outs += e.outs; bl.rbi += e.rbi;
      if (!NOT_AB.includes(e.outcome)) bl.ab++;
      if (HITS.includes(e.outcome)) { bl.h++; pl.h++; }
      if (e.outcome === '2B') bl.d++;
      if (e.outcome === '3B') bl.t++;
      if (e.outcome === 'HR') { bl.hr++; pl.hr++; }
      if (e.outcome === 'BB' || e.outcome === 'IBB') { bl.bb++; pl.bb++; }
      if (e.outcome === 'K') { bl.k++; pl.k++; }
      if (e.outcome === 'HBP') bl.hbp++;
      if (e.outcome === 'SAC') bl.sh++;
      if (e.outcome === 'SF') bl.sf++;
      if (e.outcome === 'ROE') errors[fld]++;
    } else if (e.t === 'run') {
      b(e.side, e.runnerId).r++;
      const pl = p(other(e.side), e.pitcherId);
      pl.r++;
      if (e.earned) pl.er++;
    } else if (e.t === 'steal') {
      const bl = b(e.side, e.runnerId);
      if (e.success) bl.sb++;
      else { bl.cs++; p(other(e.side), e.pitcherId).outs++; }
    } else if (e.t === 'sub') {
      if (e.kind === 'PH') b(e.side, e.inId);
      else p(e.side, e.inId);
    }
  }

  const list = <T>(m: Map<string, T>) => [...m.values()];
  const hits = (side: Side) => list(bat[side]).reduce((a, l) => a + l.h, 0);
  return {
    batting: { away: list(bat.away), home: list(bat.home) },
    pitching: { away: list(pit.away), home: list(pit.home) },
    runs: { away: s.score.away, home: s.score.home },
    hits: { away: hits('away'), home: hits('home') },
    errors,
  };
}
```

Note: the `pitch` event has no side field, so the fielding side comes from which roster holds the pitcher. Player ids are unique across teams in fixtures, and M2 must keep them unique across the league.

- [ ] **Step 4: Run tests**

Run: `npx vitest run tests/boxScore.test.ts && npm run typecheck`
Expected: 3 pass; typecheck exits 0.

- [ ] **Step 5: Commit**

```bash
git add src/baseball/boxScore.ts tests/boxScore.test.ts
git commit -m "feat: box score folded from the event stream" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 11: Save and reload, including mid at-bat

**Files:**
- Create: `src/save/save.ts`
- Test: `tests/save.test.ts`

**Interfaces:**
- Consumes: `GameState` (Task 4); `simulateGame`, `replay` (Task 9); `applyAction` (Task 8).
- Produces:
  - `SAVE_VERSION` (1)
  - `serialize(s: GameState): string`
  - `deserialize(json: string): GameState` (throws on a bad version or bad JSON)
  - `interface KV { getItem(k: string): string | null; setItem(k: string, v: string): void }`
  - `writeSave(kv: KV, s: GameState): void` (keeps the previous save as `rafters.save.prev`)
  - `readSave(kv: KV): GameState | null` (falls back to the previous save if the current one is unreadable)

The state is plain JSON, including every RNG stream's position, so `JSON.stringify` is the whole save. The two-slot write means a crash during a save never loses the last good game.

- [ ] **Step 1: Write the failing tests**

`tests/save.test.ts`:
```ts
import { serialize, deserialize, writeSave, readSave, type KV } from '../src/save/save';
import { simulateGame, replay } from '../src/baseball/sim';
import { applyAction } from '../src/baseball/game';
import { makeTeam } from '../src/baseball/fixtures';

const away = makeTeam('aw', 'Away');
const home = makeTeam('hm', 'Home');
const memoryKV = (): KV & { data: Map<string, string> } => {
  const data = new Map<string, string>();
  return { data, getItem: (k) => data.get(k) ?? null, setItem: (k, v) => void data.set(k, v) };
};

test('a game saved mid at-bat continues identically after reload', () => {
  const { state: full, actions } = simulateGame(away, home, 1924, 'midab');
  // Find a cut point inside a plate appearance (count not 0-0).
  let cut = -1;
  const probe = replay(away, home, 1924, 'midab', []);
  for (let i = 0; i < actions.length; i++) {
    applyAction(probe, actions[i]!);
    if (i > 40 && (probe.balls > 0 || probe.strikes > 0)) { cut = i + 1; break; }
  }
  expect(cut).toBeGreaterThan(0);
  const before = replay(away, home, 1924, 'midab', actions.slice(0, cut));
  const reloaded = deserialize(serialize(before));
  expect(reloaded.balls + reloaded.strikes).toBeGreaterThan(0);
  for (const a of actions.slice(cut)) applyAction(reloaded, a);
  expect(reloaded.events).toEqual(full.events);
  expect(reloaded.score).toEqual(full.score);
});

test('a save from another version is refused', () => {
  const { state } = simulateGame(away, home, 1918, 'ver');
  const bad = JSON.stringify({ ...state, version: 99 });
  expect(() => deserialize(bad)).toThrow(/version/);
  expect(() => deserialize('{not json')).toThrow();
});

test('an unreadable save falls back to the previous one', () => {
  const kv = memoryKV();
  const first = simulateGame(away, home, 1918, 'one').state;
  const second = simulateGame(away, home, 1918, 'two').state;
  writeSave(kv, first);
  writeSave(kv, second);
  expect(readSave(kv)?.score).toEqual(second.score);
  kv.data.set('rafters.save', '{corrupt');
  expect(readSave(kv)?.score).toEqual(first.score);
  expect(readSave(memoryKV())).toBeNull();
});
```

- [ ] **Step 2: Run to verify failure**

Run: `npx vitest run tests/save.test.ts`
Expected: FAIL (cannot resolve `../src/save/save`).

- [ ] **Step 3: Implement**

`src/save/save.ts`:
```ts
import type { GameState } from '../baseball/types';

export const SAVE_VERSION = 1;
const KEY = 'rafters.save';
const PREV = 'rafters.save.prev';

export interface KV { getItem(k: string): string | null; setItem(k: string, v: string): void }

export function serialize(s: GameState): string {
  return JSON.stringify(s);
}

export function deserialize(json: string): GameState {
  const s = JSON.parse(json) as Partial<GameState> | null;
  if (!s || s.version !== SAVE_VERSION) throw new Error(`unsupported save version ${String(s?.version)}`);
  return s as GameState;
}

export function writeSave(kv: KV, s: GameState): void {
  const current = kv.getItem(KEY);
  if (current !== null) kv.setItem(PREV, current);
  kv.setItem(KEY, serialize(s));
}

export function readSave(kv: KV): GameState | null {
  for (const key of [KEY, PREV]) {
    const raw = kv.getItem(key);
    if (raw === null) continue;
    try { return deserialize(raw); } catch { /* try the previous save */ }
  }
  return null;
}
```

- [ ] **Step 4: Run tests**

Run: `npx vitest run tests/save.test.ts && npm run typecheck`
Expected: 3 pass; typecheck exits 0.

- [ ] **Step 5: Commit**

```bash
git add src/save/save.ts tests/save.test.ts
git commit -m "feat: versioned saves with mid-at-bat reload and last-good fallback" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 12: Calibrate against real 1918–1930 baseball

**Files:**
- Create: `tests/calibration.test.ts`
- Modify (only as the tuning procedure directs): `src/baseball/manager.ts` (`KNOBS`), `src/baseball/bases.ts` (the `go(...)` base rates for runners scoring from second on a single and from first on a double), `scripts/build-eras.mjs` (`ROE_SHARE_OF_ERRORS`), then regenerate `src/baseball/eras.json`

**Interfaces:**
- Consumes: `simulateGame` (Task 9), `getEra` (Task 3), `makeTeam` (Task 4).
- Produces: a calibration test that fails if the engine drifts from each era's real rates, plus a printed report.

Two league-average fixture teams play 400 games in each of four seasons: 1918 (dead ball), 1920 (the shift), 1925 and 1930 (live ball). Their per-team-game rates are compared with the real league. Fixture pitchers hit like pitchers, as in the real data, so the lineup mix matches.

Tolerance bands, relative to the real value:
- Strikeouts per plate appearance: ±12%.
- Walks per plate appearance: ±12%.
- Home runs per plate appearance: ±20%.
- Runs per game: ±10%.
- Steals per game: ±25%.
- Sacrifices per game: ±25%.

- [ ] **Step 1: Write the calibration test**

`tests/calibration.test.ts`:
```ts
import { simulateGame } from '../src/baseball/sim';
import { getEra } from '../src/baseball/eras';
import { makeTeam } from '../src/baseball/fixtures';

const away = makeTeam('aw', 'Away');
const home = makeTeam('hm', 'Home');
const GAMES = 400;

function measure(year: number) {
  let pa = 0, k = 0, bb = 0, hr = 0, sh = 0, sb = 0, runs = 0, teamGames = 0;
  for (let i = 0; i < GAMES; i++) {
    const { state } = simulateGame(away, home, year, `cal-${year}-${i}`);
    teamGames += 2;
    runs += state.score.away + state.score.home;
    for (const e of state.events) {
      if (e.t === 'pa') {
        pa++;
        if (e.outcome === 'K') k++;
        if (e.outcome === 'BB' || e.outcome === 'IBB') bb++;
        if (e.outcome === 'HR') hr++;
        if (e.outcome === 'SAC') sh++;
      } else if (e.t === 'steal' && e.success) sb++;
    }
  }
  return { k: k / pa, bb: bb / pa, hr: hr / pa, rpg: runs / teamGames, sbpg: sb / teamGames, shpg: sh / teamGames };
}

function real(year: number) {
  const e = getEra(year);
  return { k: e.k, bb: e.bb, hr: e.hrPerContact * (1 - e.k - e.bb - e.hbp), rpg: e.runsPerGame, sbpg: e.sbPerGame, shpg: e.shPerGame };
}

const BANDS = { k: 0.12, bb: 0.12, hr: 0.2, rpg: 0.1, sbpg: 0.25, shpg: 0.25 } as const;

test('simulated seasons match the real league within tolerance', () => {
  const rows: Record<string, string>[] = [];
  const misses: string[] = [];
  for (const year of [1918, 1920, 1925, 1930]) {
    const sim = measure(year);
    const ref = real(year);
    for (const key of Object.keys(BANDS) as (keyof typeof BANDS)[]) {
      const rel = sim[key] / ref[key] - 1;
      rows.push({ year: String(year), stat: key, sim: sim[key].toFixed(4), real: ref[key].toFixed(4), off: `${(rel * 100).toFixed(1)}%` });
      if (Math.abs(rel) > BANDS[key]) misses.push(`${year} ${key} ${(rel * 100).toFixed(1)}%`);
    }
  }
  console.table(rows);
  expect(misses).toEqual([]);
});
```

- [ ] **Step 2: Run it and read the report**

Run: `npx vitest run tests/calibration.test.ts`
Expected: a table of 24 rows. The test passes or lists the misses. It should take under a minute (`testTimeout` is 60 s).

- [ ] **Step 3: Tune, only if there are misses, in this order**

Re-run the test after each change:
1. **Steals off:** scale `KNOBS.stealAttempt` by `real / sim` (once, then again if still outside the band).
2. **Sacrifices off:** scale `KNOBS.sacBunt` by `real / sim`, capped at 0.95.
3. **Runs off in the same direction in every year:** move both extra-base base rates in `bases.ts` together, the `0.62` (scoring from second on a single) and the `0.42` (scoring from first on a double). Move them up if runs are low and down if high, in steps of 0.03, at most ±0.12 in total.
4. **Runs still off:** change `ROE_SHARE_OF_ERRORS` in `scripts/build-eras.mjs` within 0.35–0.65, run `npm run eras`, and re-run.
5. **K, BB or HR off:** don't tune these. They come straight from the data through the solver, so a miss there is a bug. Check `paTargets` against `getEra` for an average matchup, and check how much of the lineup the pitcher's at-bats make up.

If runs miss in opposite directions in different years after step 4, stop and report the table. That means the era model needs another data field, which is a design question, not a tuning one.

Record the final knob values in a comment next to each one: `// tuned 2026-10-xx against calibration.test.ts`.

- [ ] **Step 4: Run the full suite**

Run: `npm test && npm run typecheck`
Expected: every test passes.

- [ ] **Step 5: Commit**

```bash
git add tests/calibration.test.ts src/baseball/manager.ts src/baseball/bases.ts scripts/build-eras.mjs src/baseball/eras.json
git commit -m "test: calibrate the engine against 1918-1930 league rates" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 13: The bare diamond

**Files:**
- Create: `src/ui/Diamond.tsx`, `src/ui/styles.css`
- Modify: `src/ui/App.tsx` (replace the placeholder), `src/ui/main.tsx` (import the stylesheet)

**Interfaces:**
- Consumes: `newGame`, `applyAction`, `legalActions`, `currentBatter`, `currentPitcher`, `battingSide`, `fieldingSide` (Task 8); `aiAction` (Task 9); `boxScore` (Task 10); `readSave`, `writeSave` (Task 11); `ERA_YEARS` (Task 3); `makeTeam` (Task 4).
- Produces: a playable page at `npm run dev`.

The page has:
- **Header:** the season picker and a New game button.
- **Line score** (with an X for an unplayed bottom of the 9th).
- **The diamond:** an SVG showing runners and outs.
- **The situation:** count, outs, batter vs. pitcher, and the pitcher's pitch count.
- **Controls:**
  - A pitch button for each batter action (swing away, take, protect, bunt).
  - Buttons for the other legal moves.
  - "Let the manager decide", plus sim to the end of the at-bat, the half-inning or the game.
- **A play-by-play log,** and the box score when the game ends.

It autosaves after every action, so reloading the page mid at-bat resumes at the same count. The UI has no game logic of its own: it only calls engine functions. It must work at phone width.

- [ ] **Step 1: Write the diamond**

`src/ui/Diamond.tsx`:
```tsx
import type { GameState } from '../baseball/types';

// Bases in a 200x200 box: first, second, third.
const BASES = [[165, 105], [100, 40], [35, 105]] as const;

export function Diamond({ s }: { s: GameState }) {
  const label = ['first', 'second', 'third'].filter((_, i) => s.bases[i]).join(', ') || 'bases empty';
  return (
    <svg viewBox="0 0 200 200" className="diamond" role="img" aria-label={`Runners: ${label}. ${s.outs} out.`}>
      <polygon points="100,170 165,105 100,40 35,105" className="infield" />
      {BASES.map(([x, y], i) => (
        <rect key={i} x={x - 9} y={y - 9} width={18} height={18} transform={`rotate(45 ${x} ${y})`}
          className={s.bases[i] ? 'base on' : 'base'} />
      ))}
      <polygon points="92,166 108,166 108,174 100,181 92,174" className="plate" />
      {[0, 1, 2].map((i) => <circle key={i} cx={20 + i * 16} cy={185} r={6} className={i < s.outs ? 'out on' : 'out'} />)}
    </svg>
  );
}
```

- [ ] **Step 2: Write the page**

`src/ui/App.tsx`:
```tsx
import { useState } from 'react';
import type { Action, BatterAction, GameEvent, GameState, PaOutcome, Side } from '../baseball/types';
import { applyAction, battingSide, currentBatter, currentPitcher, fieldingSide, legalActions, newGame } from '../baseball/game';
import { aiAction } from '../baseball/manager';
import { boxScore } from '../baseball/boxScore';
import { ERA_YEARS } from '../baseball/eras';
import { makeTeam } from '../baseball/fixtures';
import { readSave, writeSave, type KV } from '../save/save';
import { Diamond } from './Diamond';

const storage: KV | null = (() => { try { return window.localStorage; } catch { return null; } })();
const fresh = (year: number) => newGame(makeTeam('aw', 'Visitors'), makeTeam('hm', 'Reds'), year, `game-${Date.now()}`);
const load = (): GameState => (storage && readSave(storage)) || fresh(1918);

const SWING: Record<BatterAction, string> = { normal: 'Swing away', take: 'Take', protect: 'Protect the plate', bunt: 'Bunt' };
const PLAY: Record<PaOutcome, string> = {
  K: 'strikes out', BB: 'walks', IBB: 'is walked intentionally', HBP: 'is hit by a pitch', '1B': 'singles', '2B': 'doubles',
  '3B': 'triples', HR: 'homers', ROE: 'reaches on an error', GO: 'grounds out', GDP: 'grounds into a double play',
  FC: "reaches on a fielder's choice", FO: 'flies out', SF: 'hits a sacrifice fly', LO: 'lines out', SAC: 'sacrifices', BUNT_HIT: 'beats out a bunt',
};
const ordinal = (n: number) => `${n}${n % 10 === 1 && n !== 11 ? 'st' : n % 10 === 2 && n !== 12 ? 'nd' : n % 10 === 3 && n !== 13 ? 'rd' : 'th'}`;
const count = (s: GameState, t: GameEvent['t']) => s.events.reduce((a, e) => a + (e.t === t ? 1 : 0), 0);

export function App() {
  const [s, setS] = useState<GameState>(load);
  const [year, setYear] = useState(s.year);
  const names = new Map([...s.teams.away.players, ...s.teams.home.players].map((p) => [p.id, p.name]));
  const nm = (id: string) => names.get(id) ?? id;

  const commit = (n: GameState) => { try { if (storage) writeSave(storage, n); } catch { /* storage full or blocked */ } setS(n); };
  const run = (f: (n: GameState) => void) => { const n = structuredClone(s); f(n); commit(n); };
  const act = (a: Action) => run((n) => applyAction(n, a));
  const simUntil = (t: GameEvent['t'] | 'final') => run((n) => {
    const start = t === 'final' ? 0 : count(n, t);
    while (!n.final && (t === 'final' || count(n, t) === start)) applyAction(n, aiAction(n));
  });

  const describe = (e: GameEvent): string => {
    switch (e.t) {
      case 'pitch': return e.result === 'inPlay' ? 'In play' : `${{ ball: 'Ball', strike: 'Strike', foul: 'Foul', hbp: 'Hit by pitch' }[e.result]} (${e.balls}-${e.strikes})`;
      case 'pa': return `${nm(e.batterId)} ${PLAY[e.outcome]}${e.rbi ? ` (${e.rbi} RBI)` : ''}`;
      case 'run': return `${nm(e.runnerId)} scores`;
      case 'steal': return `${nm(e.runnerId)} ${e.success ? 'steals' : 'is caught stealing'} ${e.from === 1 ? 'second' : 'third'}`;
      case 'sub': return e.kind === 'PH' ? `${nm(e.inId)} pinch-hits for ${nm(e.outId)}` : `${nm(e.inId)} relieves ${nm(e.outId)}`;
      case 'half': return `${e.half === 'top' ? 'Top' : 'Bottom'} of the ${ordinal(e.inning)}`;
      case 'final': return `Final: ${s.teams.away.name} ${e.away}, ${s.teams.home.name} ${e.home}`;
    }
  };
  const moveText = (a: Action): string => {
    switch (a.kind) {
      case 'ibb': return 'Intentional walk';
      case 'steal': return `Steal ${a.from === 1 ? 'second' : 'third'}`;
      case 'pinchHit': return `Pinch-hit: ${nm(a.inId)}`;
      case 'changePitcher': return `Bring in ${nm(a.inId)}`;
      case 'pitch': return SWING[a.batter];
    }
  };

  const acts = legalActions(s);
  const innings = Math.max(9, s.lineScore.away.length);
  const sides: Side[] = ['away', 'home'];
  const box = s.final ? boxScore(s) : null;
  const pitcher = !s.final ? currentPitcher(s) : null;

  return (
    <main>
      <header>
        <h1>Hometown Rafters</h1>
        <label>Season <select value={year} onChange={(e) => setYear(Number(e.target.value))}>
          {ERA_YEARS.map((y) => <option key={y} value={y}>{y}</option>)}
        </select></label>
        <button onClick={() => commit(fresh(year))}>New game</button>
      </header>

      <div className="scroll">
        <table className="linescore">
          <thead><tr><th>{s.year}</th>{Array.from({ length: innings }, (_, i) => <th key={i}>{i + 1}</th>)}<th>R</th></tr></thead>
          <tbody>{sides.map((x) => (
            <tr key={x}><th>{s.teams[x].name}</th>
              {Array.from({ length: innings }, (_, i) => <td key={i}>{s.lineScore[x][i] ?? (s.final && x === 'home' && i >= s.lineScore.home.length ? 'X' : '')}</td>)}
              <td><b>{s.score[x]}</b></td></tr>
          ))}</tbody>
        </table>
      </div>

      <section className="field">
        <Diamond s={s} />
        {!s.final && pitcher && (
          <div className="situation">
            <p>{s.half === 'top' ? 'Top' : 'Bottom'} {ordinal(s.inning)} · {s.outs} out</p>
            <p className="count">{s.balls}-{s.strikes}</p>
            <p><b>{currentBatter(s).name}</b> ({s.teams[battingSide(s)].name})</p>
            <p>vs {pitcher.name} · {s.sides[fieldingSide(s)].pitchCount} pitches</p>
          </div>
        )}
      </section>

      {!s.final && (
        <section className="controls">
          <div>{acts.filter((a) => a.kind === 'pitch').map((a) => <button key={moveText(a)} onClick={() => act(a)}>{moveText(a)}</button>)}</div>
          <div>{acts.filter((a) => a.kind !== 'pitch').map((a) => <button key={moveText(a)} className="minor" onClick={() => act(a)}>{moveText(a)}</button>)}</div>
          <div>
            <button className="minor" onClick={() => run((n) => applyAction(n, aiAction(n)))}>Let the manager decide</button>
            <button className="minor" onClick={() => simUntil('pa')}>Sim at-bat</button>
            <button className="minor" onClick={() => simUntil('half')}>Sim half-inning</button>
            <button className="minor" onClick={() => simUntil('final')}>Sim game</button>
          </div>
        </section>
      )}

      <ol className="log">{s.events.slice(-14).reverse().map((e, i) => <li key={s.events.length - i}>{describe(e)}</li>)}</ol>

      {box && sides.map((x) => (
        <div key={x} className="scroll">
          <table className="box">
            <thead><tr><th>{s.teams[x].name}</th><th>AB</th><th>R</th><th>H</th><th>RBI</th><th>BB</th><th>K</th></tr></thead>
            <tbody>{box.batting[x].map((b) => <tr key={b.playerId}><th>{nm(b.playerId)}</th><td>{b.ab}</td><td>{b.r}</td><td>{b.h}</td><td>{b.rbi}</td><td>{b.bb}</td><td>{b.k}</td></tr>)}</tbody>
          </table>
          <table className="box">
            <thead><tr><th>Pitching</th><th>IP</th><th>H</th><th>R</th><th>ER</th><th>BB</th><th>K</th><th>P</th></tr></thead>
            <tbody>{box.pitching[x].map((p) => <tr key={p.playerId}><th>{nm(p.playerId)}</th><td>{Math.floor(p.outs / 3)}.{p.outs % 3}</td><td>{p.h}</td><td>{p.r}</td><td>{p.er}</td><td>{p.bb}</td><td>{p.k}</td><td>{p.pitches}</td></tr>)}</tbody>
          </table>
        </div>
      ))}
    </main>
  );
}
```

- [ ] **Step 3: Styles and stylesheet import**

`src/ui/main.tsx`:
```tsx
import { createRoot } from 'react-dom/client';
import { App } from './App';
import './styles.css';

createRoot(document.getElementById('root')!).render(<App />);
```

`src/ui/styles.css`:
```css
:root { --bg: #f4efe4; --ink: #1d1b16; --muted: #6b6457; --line: #c9bfa8; --grass: #5f7f4f; --dirt: #c79a63; --accent: #8c1d18; }
@media (prefers-color-scheme: dark) { :root { --bg: #17150f; --ink: #ece5d3; --muted: #a39a86; --line: #3d382c; --grass: #3b5232; --dirt: #7a5e3c; --accent: #d0574f; } }
* { box-sizing: border-box; }
body { margin: 0; background: var(--bg); color: var(--ink); font: 16px/1.4 Georgia, 'Times New Roman', serif; }
main { max-width: 720px; margin: 0 auto; padding: 12px 16px 48px; }
header { display: flex; flex-wrap: wrap; gap: 8px 12px; align-items: center; }
h1 { font-size: 1.4rem; margin: 0; flex: 1 1 100%; }
button, select { font: inherit; min-height: 44px; padding: 6px 12px; border: 1px solid var(--line); background: transparent; color: var(--ink); border-radius: 4px; }
button { cursor: pointer; }
button:not(.minor) { border-color: var(--accent); color: var(--accent); font-weight: bold; }
.scroll { overflow-x: auto; margin: 12px 0; }
table { border-collapse: collapse; font-variant-numeric: tabular-nums; }
th, td { padding: 4px 6px; border-bottom: 1px solid var(--line); text-align: center; }
tbody th, thead th:first-child { text-align: left; white-space: nowrap; }
.field { display: flex; flex-wrap: wrap; gap: 12px; align-items: center; }
.diamond { width: 200px; height: 200px; flex: none; }
.infield { fill: var(--dirt); stroke: var(--grass); stroke-width: 6; }
.base { fill: var(--bg); stroke: var(--ink); }
.base.on { fill: var(--accent); }
.plate { fill: var(--bg); stroke: var(--ink); }
.out { fill: none; stroke: var(--ink); }
.out.on { fill: var(--ink); }
.situation p { margin: 2px 0; }
.count { font-size: 2rem; font-weight: bold; }
.controls div { display: flex; flex-wrap: wrap; gap: 8px; margin: 8px 0; }
.log { padding-left: 20px; color: var(--muted); }
.log li:first-child { color: var(--ink); }
```

- [ ] **Step 4: Build check**

Run: `npm run build`
Expected: typecheck passes and Vite writes `dist/` without errors.

- [ ] **Step 5: Manual check in the browser**

Run: `npm run dev`, then open the printed local URL. Check that:
1. Pitches update the count, the diamond and the log.
2. A walk puts a runner on first.
3. "Sim half-inning" advances the half.
4. Reloading the page mid at-bat comes back at the same count.
5. "Sim game" ends with a box score, and the line score shows X when the home team didn't bat in the 9th.
6. At 375 px width, nothing scrolls sideways except the line-score and box tables.

- [ ] **Step 6: Commit**

```bash
git add src/ui/App.tsx src/ui/Diamond.tsx src/ui/main.tsx src/ui/styles.css
git commit -m "feat: bare diamond page with pitch-by-pitch play, sims and autosave" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
```

---
### Task 14: Finish M1

**Files:**
- Modify: `README.md` (add the M1 status section)
- Modify (docs repo): `D:\Claude Context\Hometown Rafters\HANDOFF.md` (one line pointing at the game repo and M1's state)

**Interfaces:**
- Consumes: everything above.
- Produces: a pushed, green M1, plus the numbers M2 starts from.

- [ ] **Step 1: Full verification**

Run: `cd /d/CodexWorkspaces/hometown-rafters-game && npm test && npm run build`
Expected: every test file passes (smoke, rng, eras, fixtures, matchup, pitchModel, bases, game, sim, boxScore, save, calibration), and the build succeeds. Copy the calibration table from the output.

- [ ] **Step 2: README status**

Append to `README.md`:
```markdown
## Status: M1 (baseball core) complete

- One game, simmed or played pitch by pitch, for any season 1918–1930.
- Engine: odds-ratio + log5 matchups, a count chain solved to each matchup's targets, runner movement,
  walk-offs, extra innings, steals, bunts, intentional walks, pinch hitters, pitching changes, fatigue.
- Saves: versioned JSON in browser storage, reloadable mid at-bat, last-good fallback.
- Calibration: see `tests/calibration.test.ts`; the engine matches real league rates within its bands.

Next: M2 (eight generated teams, a 154-game season, standings, transactions). Plan to be written
when M1 is accepted.
```

- [ ] **Step 3: Commit and push**

```bash
git add README.md
git commit -m "docs: M1 status" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>"
git push
```
Expected: `git status -sb` shows `main...origin/main` with nothing pending.

- [ ] **Step 4: Point the docs repo at the game**

Add this line under the "UPDATE (10/19, later)" heading in `D:\Claude Context\Hometown Rafters\HANDOFF.md`:
```markdown
M1 (baseball core) is built in `D:\CodexWorkspaces\hometown-rafters-game` (private GitHub repo walshja9/hometown-rafters-game). Next: the M2 plan.
```
Then commit and push in the docs repo:
```bash
cd "/d/Claude Context/Hometown Rafters" && git add HANDOFF.md && git commit -m "HANDOFF: M1 built" -m "Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>" && git push
```

---

## Self-review notes (written with the plan)

**Spec coverage for M1** (spec §9, §11 and §13 M1):
- **Engine design:** one pitch-level engine (Tasks 6 and 8). Log5 targets with a count model (5, 6). The two `sim_engine.py` bugs fixed (5, the K/BB/HBP cap; 8, the walk-off stop).
- **Era environments:** per season from real data (3), with calibration (12).
- **Modes:** simmed games via the AI manager (9); managed calls (8: steal, bunt, pinch hitter, pitching change, intentional walk); Moment batter actions (6, 8).
- **Rules (5, 7 and 8):**
  - runner identities and hit-type advancement with speed and arm;
  - steals and caught stealing, sacrifices, double plays and errors;
  - walk-offs and extra innings;
  - fatigue, pitchers batting, and legal substitutions.
- **Outputs:** box score from one event stream (10).
- **Determinism and modes:** seeded streams (2); same-seed and mode-equivalence tests (9).
- **Saves:** mid at-bat with last-good fallback (11). Bare diamond (13).
- **Deliberately not in M1** (listed under "Deferred"): hit-and-run, pitcher-side Moment actions, sitting on a pitch type, outs on extra-base attempts, the official-scorer walk-off hit rule, double switches, darkness ties, and the Web Worker (the spec adds it only if simulation stalls the UI).

**Type consistency check:**
- `PaTargets` and `PaKey` (Task 5) are used by `inPlayWeights` (6) and `targetsFor` (8).
- `InPlayKey` (6) is used by `PlayKind` (7).
- `PlayResult` (7) is used by `endPa` (8).
- `KV` (11) is used by the UI (13).
- `Action` and `BatterAction` (4) are used everywhere.
- `setTestDraws`, `targetsFor` and `INNINGS` are exported by Task 8 and used by its tests.
