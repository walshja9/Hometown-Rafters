# M5a: The Rest of the Era (1921–1930 deck, letters, the ending) — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking. The implementer is Codex (one dispatch per task, write-capable, cwd = the game repo, verifies with `npm test -- --configLoader native --pool threads` and `npm run build -- --configLoader native`, cannot write `.git`); Fable reviews each diff, runs `npm test` and `npm run build`, commits with explicit paths, and writes the content in Tasks 6, 7 and 8.

**Goal:** A player who starts 1918 reaches 1930 without seeing a card twice word for word, reads a letter every winter that knows what year it is, and gets an ending that tells them what they built.

**Architecture:** Content-first. Three small engine additions (two gate/sale effects and a `borrow` effect; body variants for cards and flag variants for letters; a `last_walk` flag) carry 28 new cards, 27 letter texts and 12 readouts. The ending is a pure module (`src/franchise/verdict.ts`) rendered by one new page (`EndPage`) that replaces the `over` stub and the foreclosure headline. The bots run 1918–1930 unchanged as the numbers gate.

**Tech Stack:** unchanged (React 19, TypeScript, Vite, vitest, `yaml`). No new dependencies.

**Repo:** `D:/CodexWorkspaces/hometown-rafters-game` (main at 450f31b, 499 tests green). Spec: docs repo `docs/superpowers/specs/2026-10-21-m5a-era-content-design.md`. Alex's decisions: everything the handoff lists is M5, split M5a/M5b/M5c; Fable writes all prose; local spine fixed with national headlines as weighted cards; deck shape C (spine + windowed + yearly variants); the Bandbox Collapse is foreshadowed only (closing-screen grandstand line), Gus plays through 1930.

## Global Constraints

- Hidden values (trust, culture, loyalty, infrastructure, drift, potential) are never shown as numbers; they are read back only as prose. The finances verdict's infrastructure line reads as the grandstand's condition.
- Determinism: no new `rng.league` draws in-season; card draws use `rng.story` as today.
- Save version goes 6 → 7 (new `Franchise.demandMod`). v1–v6 saves are refused; no migration.
- Every card keeps at least one option with no `spend` (checker rule). Every spine card offers one option per path (roots / ambition / money). Options with no `path` must put the safe option first (bots fall back to the first affordable option).
- New files under `content/` need `src/story/content.ts` touched in the same change or Vitest will not see them (cached `import.meta.glob`).
- Bots gate (tests/bots/bots.test.ts) must stay green: no careful bot forecloses before 1929; roots − money band ≥ 1; no club > 7 pennants (Reds ≤ 5); league RPG within 10% of the era every year; money bot 1930 cash ≥ 0; roots/m4-a deterministic. Tuning rule: ≤ 3 constants per pass, ≤ 3 passes, then a finding for Alex.
- Codex sandbox: run `npm test -- --configLoader native --pool threads` and `npm run build -- --configLoader native`; never write `.git`. The bots suite takes ~5 minutes; run it only in Task 9 and in Task 10's final check.
- Style: single-quote TS, dense JSX, existing table/`.prose`/`.boards` classes, no new abstractions.

## File map

Create:
- `src/franchise/verdict.ts` — pure functions: `baseballVerdict`, `financesVerdict`, `townVerdict`, `retrospective`, `carryForward`, `pennantYears`, `finalRecords`.
- `src/ui/EndPage.tsx` — the closing screen (and the foreclosure screen).
- `content/cards/*.md` — 28 new cards (Task 6).
- `tests/franchise/verdict.test.ts`, `tests/endPage.test.tsx`, `tests/story/coverage.test.ts`, `tests/story/variants.test.ts`.
- `docs/m5a-build/` — ledger `progress.md`, task reports, `bots-report.md`, `final-review.md`.

Modify:
- `src/save/save.ts:3`, `src/league/types.ts:41`, `src/league/league.ts:25`, `tests/league/helpers.ts:8` (+ the version tests listed in Task 1) — v7.
- `src/franchise/types.ts:146-157` — `demandMod: number`.
- `src/franchise/economy.ts:96-108` (`attendanceFor`), `:120-128` (`borrow`, reuse) — demand multiplier.
- `src/franchise/loop.ts:111-131` — `last_walk` flag in `toWalk`, `demandMod = 0` in `toWinter`.
- `src/story/types.ts` — `Effect` union (+ `demand`, `sell_best`, `borrow`), `Card.body: string | string[]`, `LetterYear.variants`.
- `src/story/scheduler.ts:87-124` (`applyEffect`), `:53-71` (`sellFavorite`, `optionProblem`), and wherever a card body is handed to the UI (`CardPage.tsx`) — variants.
- `src/story/content.ts:165-181` — body variant split; a comment touch whenever content is added.
- `src/story/check.ts` — variant rules, `sell_best`/`borrow` effect validation, letter variants.
- `src/ui/App.tsx:67-71` — `EndPage` replaces the `over` early return.
- `src/ui/OfficePage.tsx:21-22` — letter flag variants.
- `content/letters.yaml`, `content/readouts.yaml`, the four yearly card files (Tasks 7, 8, 6).
- `README.md`, docs repo `HANDOFF.md`, this plan's Execution notes (Task 10).

---

### Task 1: Save v7 and `demandMod`

**Files:**
- Modify: `src/save/save.ts:3` (`SAVE_VERSION = 7`), `src/league/types.ts:41` (`version: 7`), `src/league/league.ts:25` (`version: 7`), `tests/league/helpers.ts:8` (`version: 7`).
- Modify: `src/franchise/types.ts:146-157` — add `demandMod: number;` to `Franchise` with the comment `// this year's gate multiplier from cards (1 + demandMod); reset each winter`.
- Modify: `src/league/league.ts` where the fresh `franchise` object is built — `demandMod: 0`; `tests/league/helpers.ts` `bare()` franchise — `demandMod: 0`.
- Test: `tests/save.test.ts:27-34` (refuse 1–6, `'{"version":7}'` throws /missing/), `tests/franchise/state.test.ts:16, :97`, `tests/league/rest.test.ts:29-30` ('a new league is version 7'); `tests/ui.test.tsx:55, :85` keep their version-2 saves.

**Interfaces:** Produces `L.franchise.demandMod: number` (default 0) for Tasks 2 and 9.

- [ ] **Step 1:** Update the four version sites and the tests above; add `demandMod` to the type, `newLeague` and `bare()`.
- [ ] **Step 2:** Run `npm test -- --configLoader native --pool threads` → expect 499 passing (version tests updated, nothing else moves).
- [ ] **Step 3:** Report; Fable commits `save: v7, Franchise.demandMod`.

---

### Task 2: Effects `demand`, `sell_best`, `borrow`; the gate multiplier

**Files:**
- Modify: `src/story/types.ts` `Effect` union — add
  ```ts
  | { demand: { by: number } }
  | { sell_best: { under?: number; price?: number } }
  | { borrow: number }
  ```
- Modify: `src/franchise/economy.ts:102-107` `attendanceFor` — Reds-only multiplier:
  ```ts
  const demand = reds ? 1 + f.demandMod : 1;
  return Math.min(capacity, Math.round(BASE_DRAW[c.tier] * growth * price * loyalty * results * sunday * demand));
  ```
- Modify: `src/franchise/loop.ts` `toWinter` — after `L.franchise.salaries = {};` add `L.franchise.demandMod = 0;`.
- Modify: `src/story/scheduler.ts:87-124` `applyEffect` — three branches:
  ```ts
  else if ('demand' in e) L.franchise.demandMod += e.demand.by;
  else if ('sell_best' in e) sellBest(L, e.sell_best);
  else if ('borrow' in e) { const v = borrow(L, e.borrow); if (!v.ok) throw new Error(v.reason); }
  ```
  and the helper beside `sellFavorite` (`scheduler.ts:53-64`), same richest-club/emergency/atAsk pattern but any position and an age filter, refusing a sale that leaves the Reds short:
  ```ts
  function sellBest(L: LeagueState, o: { under?: number; price?: number }): boolean {
    const me = L.clubs[L.userClub]!;
    const richest = Object.values(L.clubs).filter((c) => c.id !== L.userClub).sort((a, b) => b.cash - a.cash)[0];
    if (!richest) return false;
    const men = me.roster.filter((p) => !p.untouchable && (o.under === undefined || p.age < o.under)).sort((a, b) => value(b) - value(a));
    for (const p of men) {
      if (rosterProblem(me.roster.filter((q) => q.id !== p.id))) continue;
      const before = me.cash;
      if (sellPlayer(L, richest.id, p.id, true, true).ok) {
        if (o.price !== undefined) { const paid = me.cash - before; me.cash += o.price - paid; L.franchise.books.sales += o.price - paid; richest.cash += paid - o.price; }
        if (lineupProblem(me, L.day)) autoLineup(me, L.day);
        return true;
      }
    }
    return false;
  }
  ```
  `rosterProblem` is exported from `src/league/transactions.ts:28`; import it. `borrow` is `src/franchise/economy.ts:120` (winter-only: the only `borrow` card is a winter card; `optionProblem` must surface its reason).
- Modify: `src/story/scheduler.ts:66-71` `optionProblem` — pre-validate `sell_best` with `sellBest(structuredClone(L), o)` → `'The Reds have no man they can spare.'`; pre-validate `borrow` with `borrow(structuredClone(L), n)` → its reason.
- Modify: `src/story/check.ts` — accept the three effect kinds; `demand.by` must be a number in [-0.5, 0.5]; `borrow` a positive integer ≤ 10000.
- Test: `tests/story/scheduler.test.ts` (append) and `tests/franchise/economy.test.ts` (append).

**Interfaces:** Consumes `demandMod` (Task 1). Produces the three effects for Task 6's cards.

- [ ] **Step 1: Failing tests.** In `tests/story/scheduler.test.ts`:
  ```ts
  test('demand adds to this year\'s gate multiplier and resets at winter', () => {
    const L = newLeague('demand'); L.phase = 'season';
    applyEffect(L, { demand: { by: -0.2 } });
    expect(L.franchise.demandMod).toBeCloseTo(-0.2);
    applyEffect(L, { demand: { by: 0.05 } });
    expect(L.franchise.demandMod).toBeCloseTo(-0.15);
  });
  test('sell_best sells the most valuable man under the age and never leaves the Reds short', () => {
    const L = newLeague('sell-best'); const me = L.clubs.RI!;
    const young = me.roster.filter((p) => !p.untouchable && p.age < 27).sort((a, b) => value(b) - value(a))[0]!;
    applyEffect(L, { sell_best: { under: 27 } });
    expect(me.roster.some((p) => p.id === young.id)).toBe(false);
    expect(rosterProblem(me.roster)).toBeNull();
  });
  test('sell_best at a fixed price books that price', () => {
    const L = newLeague('sell-best-price'); const me = L.clubs.RI!; const cash = me.cash;
    applyEffect(L, { sell_best: { price: 400 } });
    expect(me.cash).toBe(cash + 400);
    expect(L.franchise.books.sales).toBe(400);
  });
  test('borrow in winter adds a loan; optionProblem reports it out of season', () => {
    const L = newLeague('borrow'); L.phase = 'winter';
    applyEffect(L, { borrow: 2000 });
    expect(L.franchise.loans.at(-1)).toEqual({ principal: 2000, year: L.year });
    L.phase = 'season';
    expect(optionProblem(L, { id: 'x', text: 'x', effects: [{ borrow: 2000 }] })).toMatch(/winter/i);
  });
  ```
  In `tests/franchise/economy.test.ts`, next to the formula test at :86:
  ```ts
  test('demandMod scales the Reds gate only', () => {
    const L = newLeague('demand-gate'); const date = L.schedule[0]!.date;
    const base = attendanceFor(L, 'RI', date), away = attendanceFor(L, 'STL', date);
    L.franchise.demandMod = -0.25;
    expect(attendanceFor(L, 'RI', date)).toBe(Math.round(base * 0.75));
    expect(attendanceFor(L, 'STL', date)).toBe(away);
  });
  ```
  (If `attendanceFor` caps at capacity for the seed, pick a seed or a weekday date where it does not; state which in the report.)
- [ ] **Step 2:** Run → the new tests fail (unknown effect kinds / `demandMod` unchanged).
- [ ] **Step 3:** Implement as above.
- [ ] **Step 4:** Run `npm test -- --configLoader native --pool threads` → all green.
- [ ] **Step 5:** Report; Fable commits `effects: demand, sell_best, borrow; Reds gate multiplier`.

---

### Task 3: Card body variants and letter flag variants

**Files:**
- Modify: `src/story/types.ts` — `Card.body: string | string[]`; `LetterYear` gains `variants?: { flag: string; even: string; warm: string; cold: string }[]`.
- Modify: `src/story/content.ts:179-181` `parseCard` — split the body on a line that is exactly `===`:
  ```ts
  export function parseCard(text: string, file: string): Card {
    const { data, body } = frontmatter(text, file);
    const parts = body.split(/\r?\n===\r?\n/).map((s) => s.trim());
    return { ...data, body: parts.length > 1 ? parts : body } as unknown as Card;
  }
  ```
- Create: `export function cardBody(card: Card, year: number): string` in `src/story/scheduler.ts` (beside `deal`): `Array.isArray(card.body) ? card.body[year % card.body.length]! : card.body`. Use it in `src/ui/CardPage.tsx` wherever `card.body` is rendered, and in the bots' `choose` path if the body is read there (it is not; confirm).
- Modify: `src/story/check.ts` — every body variant non-empty; a `repeat: never` card with variants is an error (`variants only on yearly cards`); letter `variants[].flag` must be a flag some card sets (same rule as `requires`), and each variant has all three tones.
- Modify: `src/ui/OfficePage.tsx:21-22` — pick the letter text:
  ```ts
  const yearEntry = LETTERS.years.find((y) => y.year === L.year);
  const variant = yearEntry?.variants?.find((v) => L.story.flags[v.flag]);
  const text = (variant ?? yearEntry?.text)?.[info.tone] ?? LETTERS.fallback[info.tone];
  ```
- Test: `tests/story/variants.test.ts` (new).

**Interfaces:** Produces `cardBody(card, year)` and letter `variants` for Tasks 6 and 7.

- [ ] **Step 1: Failing tests** in `tests/story/variants.test.ts`:
  ```ts
  import { parseCard } from '../../src/story/content';
  import { cardBody } from '../../src/story/scheduler';
  import { checkContent } from '../../src/story/check';
  const md = (body: string) => `---\nid: v\nyears: [1918, 1930]\nphase: winter\npriority: 10\nrepeat: yearly\noptions:\n  - id: a\n    text: A.\n    effects: []\n---\n${body}`;
  test('a body split by === becomes variants picked by year', () => {
    const c = parseCard(md('one\n===\ntwo\n===\nthree'), 'v.md');
    expect(c.body).toEqual(['one', 'two', 'three']);
    expect(cardBody(c, 1921)).toBe('two'); // 1921 % 3 = 1
    expect(cardBody(parseCard(md('only'), 'v.md'), 1921)).toBe('only');
  });
  test('checker rejects an empty variant and variants on a one-time card', () => {
    const empty = parseCard(md('one\n===\n'), 'v.md');
    expect(checkContent([empty], [], [], LETTERS, STORY_GAMES, NOTICES).join()).toMatch(/variant/);
    const once = { ...parseCard(md('one\n===\ntwo'), 'v.md'), repeat: 'never' as const };
    expect(checkContent([once], [], [], LETTERS, STORY_GAMES, NOTICES).join()).toMatch(/yearly/);
  });
  test('a letter variant is chosen by flag before tone', () => {
    // build a LETTERS-shaped object with a 1927 variant on flag boston_sold and assert the OfficePage picks it:
    // renderToString(<OfficePage L={L} run={run} pick={() => {}} />) with L.year = 1927, L.phase = 'winter', L.story.flags.boston_sold = true
    // expect(html).toContain('<variant marker text>')
  });
  ```
  (Import `LETTERS`, `STORY_GAMES`, `NOTICES` from `src/story/content`; for the letter test, mutate `LETTERS.years` in `beforeEach`/`afterEach` the way `tests/story/readouts.test.ts` splices `READOUTS`.)
- [ ] **Step 2:** Run → fail.
- [ ] **Step 3:** Implement.
- [ ] **Step 4:** Run → green; `npm run build -- --configLoader native` → clean.
- [ ] **Step 5:** Report; Fable commits `content: card body variants, letter flag variants`.

---

### Task 4: `last_walk` and `verdict.ts`

**Files:**
- Modify: `src/franchise/loop.ts:111-115` `toWalk` — before `L.story.gusLine = readout(L, 'gus')`: `if (L.year >= LAST_YEAR) L.story.flags.last_walk = true;` (`LAST_YEAR` from `src/league/league.ts:62`).
- Create: `src/franchise/verdict.ts`.
- Test: `tests/franchise/verdict.test.ts`.

**Interfaces:** Produces, for Task 5:
```ts
export interface YearLine { year: number; w: number; l: number; pennant: string }
export function finalRecords(L: LeagueState): YearLine[]          // L.history plus the live 1930 row from standings(L) when L.history lacks L.year
export function pennantYears(L: LeagueState): number[]
export function baseballVerdict(L: LeagueState): string           // one paragraph
export function financesVerdict(L: LeagueState): { line: string; ok: boolean }[]  // four lines, in order
export function townVerdict(L: LeagueState): string               // one paragraph
export function retrospective(L: LeagueState): string             // the column, paragraphs joined by '\n\n'
export function carryForward(L: LeagueState): string[]            // one line each: cash/debt, core, Gus, path, grandstand
```

- [ ] **Step 1: Failing tests.** Build states with `newLeague('verdict')`, then set `L.year = 1930`, `L.phase = 'walk'`, push `L.history` rows by hand (`{ year, pennant, userWins, userLosses }`), set flags, cash, loans, hidden values, and assert:
  ```ts
  test('baseball: four registers', () => { /* 3 pennants → /dynasty|three pennants/i; 1 → /a pennant/; 0 with 7 winning seasons → /honest|winning seasons/; 0 with 3 → /second division|also-ran/ */ });
  test('finances: each line passes and fails', () => { /* loans [] vs [{principal: 500}]; roster legal vs 4 pitchers; infrastructure 45 vs 30; cash 100 vs loans 2000 cash 1000 */ });
  test('town names the leading path', () => { /* drift roots 40 → /valley/; ambition → /reached/; money → /books/ */ });
  test('retrospective names pennant years and three memorable men', () => { /* seed L.seasons for 1921-1925 with ovr for three ids on the RI roster; expect their names in the column; expect '1924' when history[1924].pennant === 'RI' */ });
  test('carryForward grandstand line by infrastructure', () => { /* 45 → /sound/; 35 → /third-base section/; 20 → /should not be used/ */ });
  test('finalRecords appends the live 1930 row', () => { /* history to 1929, results for 1930 → last row year 1930 */ });
  ```
- [ ] **Step 2:** Run → fail (module missing).
- [ ] **Step 3: Implement `verdict.ts`.** Rules and prose templates (exact strings; `{...}` are substitutions):
  - `finalRecords`: `L.history` rows mapped to `YearLine`; if no row has `year === L.year`, append from `standings(L)`: `{ year: L.year, w: me.w, l: me.l, pennant: standings(L)[0].id }`.
  - `pennantYears`: years where `pennant === L.userClub`.
  - `baseballVerdict`: `p = pennantYears.length`, `winning = records.filter(r => r.w > r.l).length`, `years = records.length`:
    - p ≥ 3: `Three pennants or more in {years} seasons ({list}). The Reds were the club the valley measured the others by.` (use the actual count word: `Three`, `Four`…; `{list}` the years joined by ', ' with ' and ' before the last).
    - p 1–2: `A pennant in {list}. The flag flew over the third-base boards, and the men who were there still say which day it was.` (two: `Two pennants, {list}.` then the same second sentence with `flags flew`).
    - p 0, winning ≥ 6: `No pennant, but {winning} winning seasons in {years}. An honest club that played above its town most years.`
    - else: `No pennant and {winning} winning seasons in {years}. The Reds were a second-division club that the valley kept anyway.`
  - `financesVerdict` (four `{ line, ok }` in this order):
    1. `ok = loans.every(l => l.principal <= 0) && L.story.obligations.length === 0` → `The notes are paid and nothing is owed.` / `A note is still owed at the bank.` (if an obligation remains: `A promise to the neighborhood is still open.`).
    2. `ok = rosterProblem(me.roster) === null` → `A legal roster of {n} men.` / `The roster is short: {problem}.`
    3. `ok = infrastructure >= 40` → `The grandstand is sound.` / `The grandstand is not sound.`
    4. `net = me.cash - sum(principal)`; `ok = net > 0` → `Cash on hand clears the debt by ${net}.` / `The debt exceeds the cash by ${-net}.`
  - `townVerdict`: lead path = `Object.entries(h.drift).sort((a,b)=>b[1]-a[1])[0][0]` (roots / ambition / money). Opening sentence by path: roots → `The club that stayed the valley's.`; ambition → `The club that reached.`; money → `The club that kept its books.` Then loyalty band (≥60 `The boards still fill from the top down, mill families first.` / 40–59 `The boards fill on Saturdays and thin on Thursdays.` / <40 `The boards are half empty on a Saturday.`), then trust band via the existing tier (`h.trust >= 60` `Gus still calls the kids "kid."` / ≥40 `Gus plays short and says less than he did.` / else `Gus plays short, and that is all he does here.`), then culture (≥55 `The clubhouse is a place men want to be.` / <45 `The clubhouse is a place men pass through.` / else nothing). Join with spaces.
  - `retrospective`: paragraphs:
    1. `PROVIDENCE JOURNAL — THIRTEEN SEASONS AT THE BANDBOX` (use `{years.length}` as a word if fewer: `NINE SEASONS…` on an early end).
    2. Pennants: if any: `The Reds won the pennant in {list}.` else `The Reds never won the pennant.` Then best year `{w}-{l} in {year}` and worst `{w}-{l} in {year}`: `Their best year was {bw}-{bl} in {by}; their worst, {ww}-{wl} in {wy}.`
    3. Memorable men: players with ≥5 entries across `L.seasons` whose id was on the RI roster in those years (approximate: `L.seasons[y][id].ovr !== undefined` counts as a rostered year), ranked by mean `ovr`, top three; names joined: `The men the town will remember: {names}.` If none qualify: `No man stayed long enough to be remembered by the paper, which is its own story.`
    4. Money: `The club began with ${cash1918} and ends with ${cash} on hand and ${debt} owed.` (cash1918 = `COALITION_SUPPORT`-era starting cash from `newLeague`; read `L.franchise.priceHistory.length` for seasons and use the constant `STARTING_CASH` if one exists in `league.ts`, else 0 and say so in the report).
    5. Remembered choices, one sentence each when the flag is set, in this order: `strike_park` → `In 1922 the park was open to the strikers.`; `affiliated` → `The club signed with a big-league parent and sent its young men up the line.`; `boston_sold` → `The town's best man went to Boston for money.`; `boston_kept` → `Boston asked for the town's best man and did not get him.`; `rally` → `In 1928 the parish held its rally on the infield.`; `fed_line` → `In 1930 the bread line at the gate was fed from the concession stand.`; `moved_line` → `In 1930 the bread line was moved off the lot.` Omit the paragraph if no flag is set.
  - `carryForward`: five strings:
    1. `${cash} on hand, ${debt} owed.`
    2. `The core: {top five roster names by value(), joined}.`
    3. `Gus Almeida, 30, at short.` (find the untouchable man; if he is gone, `Gus Almeida is no longer with the club.`)
    4. `The path: {roots: the valley's club | ambition: the club that reached | money: the club that kept its books}.`
    5. infrastructure ≥ 40 → `The grandstand is sound.`; 25–39 → `The third-base section of the grandstand wants looking at.`; <25 → `The grandstand should not be used until it is seen to.`
  - Money format: `'$' + n.toLocaleString('en-US')` (same as BooksPage).
- [ ] **Step 4:** Run → green.
- [ ] **Step 5:** Report; Fable commits `ending: last_walk flag, verdict module`.

---

### Task 5: `EndPage`

**Files:**
- Create: `src/ui/EndPage.tsx` — props `{ L: LeagueState; restart: () => void; saveError: boolean }`.
- Modify: `src/ui/App.tsx:67-71` — replace the early return's body with `<EndPage L={L} restart={restart} saveError={saveError} />` (keep the `<main>` wrapper and the early return).
- Test: `tests/endPage.test.tsx` (renderToString, like `tests/playerPage.test.tsx`).

**Interfaces:** Consumes Task 4's functions.

- [ ] **Step 1: Failing tests:** a 1930 state renders `<h1>` `The end of the era` (or `The club has been foreclosed.` when `flags.foreclosed`), the three verdict headings `Baseball`, `Finances`, `The town`, four finances lines with a pass/fail mark (`✓`/`✗` as text), the retrospective paragraphs in `.prose`, the carry-forward list under `What carries into 1931`, and a `New league` button. The foreclosure case omits the carry-forward section and says `The run ended in {year}.`
- [ ] **Step 2:** Run → fail.
- [ ] **Step 3: Implement.** Layout: `<section>` → `<h1>` → `<h2>Baseball</h2><p className="prose">{baseballVerdict(L)}</p>` → `<h2>Finances</h2><ul>{financesVerdict(L).map(f => <li>{f.ok ? '✓' : '✗'} {f.line}</li>)}</ul>` → `<h2>The town</h2><p className="prose">` → `<h2>The paper</h2><p className="prose">{retrospective(L)}</p>` → (not foreclosed) `<h2>What carries into 1931</h2><ul>{carryForward(L).map(...)}</ul>` → `<p><button onClick={restart}>New league</button></p>` → the existing `saveError` warning.
- [ ] **Step 4:** Run → green; build clean.
- [ ] **Step 5:** Report; Fable commits `ending: EndPage`.

---

### Task 6: The deck (Fable writes, Codex wires and tests)

**Files:**
- Create (Fable): 28 files in `content/cards/`: `short-time.md`, `the-strike-year.md`, `going-south.md`, `the-quota.md`, `on-the-air.md`, `the-affiliate.md`, `the-call-up.md`, `the-boston-offer.md`, `the-boston-return.md`, `the-smith-year.md`, `the-sale.md`, `the-line.md`; `parish-feast.md`, `grounds-boy.md`, `carvalho-concession.md`, `the-portuguese-paper.md`, `morelli-protection.md`, `the-flood.md`, `the-mill-nine.md`, `russo-uniforms.md`; `black-sox-verdict.md`, `radio-craze.md`, `ruth-sixty.md`, `lindbergh-day.md`, `the-scopes-summer.md`, `barnstormers.md`, `florida-bust.md`, `crash-week.md`.
- Modify (Fable): `the-grounds-man.md`, `opening-day-band.md`, `thin-week.md`, `pennant-winter.md` — four `===` body variants each.
- Modify (Fable): `src/story/content.ts` comment line 203 (touch).
- Create (Codex): `tests/story/coverage.test.ts`.

Frontmatter per spec §2.2–2.4 (ids, years, phase, `fixed: true` + `priority: 100` on the ten spine cards, `repeat: never`, `requires`, options with `path` and `effects`, `followup`). `the-call-up`: `phase: winter`, `years: [1927, 1930]`, `repeat: yearly`, `requires: { flag: [affiliated] }`. `the-boston-return`: followup target only (excluded from the weighted draw automatically). `crash-week`: `years: [1929, 1929]`, `phase: block`, `fixed: true`, `priority: 90` (block2 of 1929; the-sale is 1929 winter so they do not collide). Flags set by the new cards (all must be reused by Task 7/8 or `requires` somewhere, or the checker is fine either way): `strike_park`, `strike_yard`, `affiliated`, `boston_matched`, `boston_sold`, `boston_kept`, `rally`, `bought_share`, `outside_owner`, `coalition_gone`, `fed_line`, `free_saturdays`, `moved_line`, `radio`, `radio_sundays`, `morelli`, `carvalho_stand`, `grounds_boy_signed`.

- [ ] **Step 1 (Fable):** Write the 12 spine/support cards. Voice check against `the-lively-ball.md` and `the-strike.md`. Each spine option's `result` is one paragraph.
- [ ] **Step 2 (Fable):** Write the 16 windowed cards and the four yearly variant bodies. Touch `content.ts`.
- [ ] **Step 3 (Codex): coverage test** `tests/story/coverage.test.ts`:
  ```ts
  import { CARDS } from '../../src/story/content';
  import { eligible } from '../../src/story/scheduler';
  import { newLeague } from '../../src/league/league';
  const SLOTS = ['winter', 'opening', 'block1', 'block2'] as const;
  test('every year and slot from 1921 to 1930 has at least four eligible cards on a fresh run', () => {
    const L = newLeague('coverage');
    for (let year = 1921; year <= 1930; year++) for (const slot of SLOTS) {
      L.year = year; L.story.flags = {}; L.story.choices = [];
      const n = CARDS.filter((c) => eligible(L, c, slot)).length;
      expect(n, `${year} ${slot}`).toBeGreaterThanOrEqual(4);
    }
  });
  test('each spine card is eligible in its year and slot and nowhere else', () => {
    const spine: [string, number, string][] = [['short-time', 1921, 'winter'], ['the-strike-year', 1922, 'block1'], ['going-south', 1923, 'block2'], ['the-quota', 1924, 'opening'], ['on-the-air', 1925, 'winter'], ['the-affiliate', 1926, 'winter'], ['the-boston-offer', 1927, 'block1'], ['the-smith-year', 1928, 'block2'], ['the-sale', 1929, 'winter'], ['the-line', 1930, 'block1']];
    // for each: eligible at (year, slot) with the chain's prerequisite flags set (going-south and the-line are followups: assert via L.story.followups instead), not eligible at year-1 or year+1
  });
  ```
  (Adapt `eligible`'s real signature from `scheduler.ts:15-30`; block cards are eligible in both block slots, so the spine's `block1`/`block2` placement is by the plan's dealing order, not the schema: for those, assert eligibility in the year only. State in the report if `eligible` takes a slot.)
- [ ] **Step 4:** `npm test` → green including `checkContent === []`; fix any checker complaint in content (Fable) or checker (Codex).
- [ ] **Step 5:** Report; Fable commits `content: the 1921-1930 deck`.

---

### Task 7: Letters 1921–1928 and 1930 (Fable)

**Files:** `content/letters.yaml` — add `text: { even, warm, cold }` for 1921–1928 and 1930 (keep `support` values; 1930 `support: 0`); 1927 adds `variants: [{ flag: boston_sold, … }, { flag: boston_matched, … }]`. Placeholders `{net}`, `{support}`, `{cash}` as today. Year beats per spec §3. Letterhead `PAWTUCKET MILL COALITION` through 1928; 1930's letterhead is `CARVALHO & SONS, BAKERS` (support 0; Carvalho's bread). Test: the existing content test; add in `tests/story/content.test.ts` one assertion that every year 1918–1930 has `text` with three tones.

- [ ] **Step 1:** Write the 27 texts (+ 6 variant texts).
- [ ] **Step 2:** `npm test` → green.
- [ ] **Step 3:** Commit `content: letters 1921-1930`.

---

### Task 8: The last walk readouts (Fable)

**Files:** `content/readouts.yaml` — 12 entries, `years: [1930, 1930]`, each `when` including `flag: last_walk`:
- gate ×4: `gate_family` + loyalty ≥ 50 → the Arrudas, still at the gate; `gate_family` + loyalty < 50; no `gate_family` + `fed_line`; default (mill families, the line).
- concession ×4: `carvalho_stand`; `chourico`; `draper_right_field` or `draper_sign`; default.
- gus ×4: trust ≥ 60; 40–59; < 40; `moved_line` (overrides the band: specificity 2 keys + flag = 3).
Specificity: the band entries today have 2 keys; last-walk entries carry `flag: last_walk` plus their keys so they win in 1930 and never match before.
Test: `tests/story/readouts.test.ts` — one test: in 1930 with `last_walk` set, `walk(L).gus` differs from 1929's text for the same trust.

- [ ] **Step 1:** Write the 12 entries. **Step 2:** `npm test` → green. **Step 3:** Commit `content: the last walk`.

---

### Task 9: The bots gate and tuning (Codex)

**Files:** `tests/bots/bots.test.ts` — report path `docs/m5a-build/bots-report.md` (and `mkdirSync('docs/m5a-build')`); `tests/bots/policies.ts` — no change unless a new card has no affordable `path` option for a policy (then `byPath` throws: fix the card, not the bot). Constants (only if the gate is red): `src/franchise/economy.ts` (`DEMAND_GROWTH`, the `0.85` post-1928 factor), the `demand.by` values in the 1930 cards.

- [ ] **Step 1:** `npm run bots` (≈5 min). Read `docs/m5a-build/bots-report.md`.
- [ ] **Step 2:** If any gate fails: ≤ 3 constants per pass, ≤ 3 passes, rerun. Record every change in the task report. Expected pressure: `the-line`'s `demand -0.20` on the money bot's 1930; if the money bot ends negative, the first knob is that value, the second the 1921 `short-time` cut.
- [ ] **Step 3:** Report with the table; Fable commits `bots: m5a gate` and the report.

---

### Task 10: Review, docs, handoff

**Files:** game repo `README.md` (M5a section: the deck, letters, the ending, `demandMod`, save v7; M5b/M5c carry-overs), `docs/m5a-build/` (`progress.md`, task reports, `final-review.md`); docs repo `HANDOFF.md` and this plan's Execution notes.

- [ ] **Step 1:** Codex adversarial review of the whole branch (`adversarial-review --wait --base 450f31b --scope branch`), focus: `sell_best` leaving an illegal roster or selling Gus; `borrow` outside winter; `demandMod` leaking across years or into AI books; variants on one-time cards; last-walk readouts matching before 1930; verdicts on a foreclosed mid-run state (no 1930 row, history short); the retrospective with fewer than three qualifying men; EndPage with empty loans/obligations. Fix every finding in one wave.
- [ ] **Step 2:** README, ledger, HANDOFF "Latest", Execution notes.
- [ ] **Step 3:** Commit and push both repos.

## Acceptance (the gate, spec §7)

Codex half: `npm test` green including `tests/bots`; `docs/m5a-build/bots-report.md` shows every gate passing; build clean. Alex half: a new league (v6 saves refused), 1918–1930 played or simmed: the spine fires in order; no card repeats word for word; the 1927 letter reads the Boston choice; the last walk revisits the 1918 choices; the three verdicts and the column read true to the run; the grandstand line lands. The human question: does the ending feel earned.

## Execution notes

(Filled in during the build: every plan correction, in the order found.)

1. **Planning:** spec §4.6 named an `affiliate` franchise field; the plan uses the flag `affiliated` instead (no new state beyond `demandMod`). Spec §2.1's `demand` is Reds-only in `attendanceFor` (every card describes the Reds' gate). The 1929 sale's note uses a new `borrow` effect because `borrow()` is winter-only and effect-driven loans did not exist.
