# M5a: the rest of the era (content), design

Date: 2026-10-21. Status: approved by Alex in conversation; written for the M5a plan.

M5 ("the rest of the era", era-1 spec §13) is split in three: **M5a** the content (this spec), **M5b** the mechanics (W/L decisions, hit-and-run, double switches, a light injury model, prices revealing potential, file export), **M5c** the period look and the color line content. Each gets its own spec, plan and gate. M5a goes first because the ending reads flags the deck sets and the bots gate every tuning change against the full 1918–1930 run.

## 1. Goal

A player who starts 1918 reaches 1930 without seeing a card twice word for word, reads a letter every winter that knows what year it is, and gets an ending that tells them what they built. Alex's M4 notes: "scenarios are repetitive" (18 cards, 14 one-time, deck spent by season 3); the end screen is a stub ("The slice ends in 1930.").

Out of scope: everything in M5b and M5c; the Bandbox Collapse (bible: early 1930s, era 2; the spec keeps Gus "30 and still playing shortstop" through 1930); playoffs; the 1923/1928 no-Sunday schedule quirk.

## 2. The deck

### 2.1 Shape
48 cards: the 18 existing, 10 fixed spine cards (one a year 1921–1930), 16 windowed one-time cards, and year-variant text on the 4 yearly cards. Four draw slots a year (winter, opening, block1 day 52, block2 day 104) as today. Every year 1921–1930 has at least four eligible cards on a fresh run, six or more in most; a static test enforces four.

Effects use the existing vocabulary (`spend`, `earn`, `adjust`, `set_flag`, `clear_flag`, `followup`, `promise`, `exploit`, `sign`, `sell_favorite`, `concession_deal`, `landmark`) plus two new ones:
- `demand: { by: number }` — adds to `L.franchise.demandMod` for the current year (a gate multiplier applied in the attendance model; cleared in `toWinter`). Range used by content: -0.25 to +0.10.
- `sell_best: { under?: number, price?: number }` — sells the highest-`value()` man on the Reds (optionally only among men under the age) to the named partner for `price` (default `value()`), booked like `sell_favorite`. Refused (option unavailable with a reason) if it would leave fewer than the roster minimum.

Every option has a `path` (roots / ambition / money) as today; each spine card offers one option per path.

### 2.2 The spine (fixed, priority 100, `repeat: never`)

| Year, slot | id | Premise | roots | ambition | money |
|---|---|---|---|---|---|
| 1921 winter | short-time | Mills on half weeks; Ashworth wants payroll cut 10% | hold pay, eat the loss (`spend` ~400, loyalty +2) | sell a man to cover it (`sell_best`) | cut it (`exploit`, culture -3, `earn` ~600) |
| 1922 block1 | the-strike-year | The nine-month textile strike | open the park to union meetings (`set_flag` strike_park, loyalty +3, `landmark` roots) | stay out of it | rent the lot to the mill as a yard (`earn` 500, `exploit`) |
| 1923 block2 | going-south | First Blackstone mill closes for good; followup of the-strike-year | sign the foreman's son (`sign` local, age 19) | buy the mill nine's best man cheap (`spend` ~300, roster add via `sign` mean high) | nothing; the boards thin (`demand` -0.10) |
| 1924 opening | the-quota | Immigration law shuts the Azores | fund the parish appeal (`spend` 250, `landmark` roots, loyalty +2) | hire the stranded men as grounds crew (`spend` 150, culture +2, infrastructure +3) | "not the club's business" (loyalty -3, trust -3 via `adjust`) |
| 1925 winter | on-the-air | A Providence station wants the games | refuse, protect the gate (loyalty +1) | Sundays only: half the fee (`earn` 300, `demand` -0.05, `set_flag` radio_sundays) | take the fee (`earn` 700, `demand` -0.15, `set_flag` radio) |
| 1926 winter | the-affiliate | A big-league club offers a working agreement | refuse (culture +1) | sell them one man outright (`sell_best` under 27) | accept: lump now, a man each winter (`earn` 1500, `set_flag` affiliated, `followup` the-call-up in 1) |
| 1927 block1 | the-boston-offer | Your best man is tempted to Boston | promise him next year (`promise` id boston, due 1, kept if flag `boston_kept`, which the 1928 return sets when he is still rostered) | match the money (`spend` = 25% of his salary, `set_flag` boston_matched) | let him go (`sell_best`, `set_flag` boston_sold) |
| 1928 block2 | the-smith-year | Parish wants a rally at the park; mill owners don't | host it (`landmark` roots, loyalty +3, `set_flag` rally) | refuse both (culture -1) | sell both sides a night (`earn` 400, trust -2) |
| 1929 winter | the-sale | Ashworth sells; the buyer wants the lot, the bank the note | buy his share on a loan (`set_flag` bought_share, loan via existing loan machinery, `spend` 0, a 2,000 note at the books' rate) | let the buyer in (`earn` 1000, `set_flag` outside_owner, trust -3) | let the Coalition dissolve (`set_flag` coalition_gone) |
| 1930 block1 | the-line | A bread line forms by the gate; followup of the-sale | feed it from concessions (`spend` 200, loyalty +4, `set_flag` fed_line) | nothing on Saturdays: free gate (`demand` -0.20, loyalty +2, `set_flag` free_saturdays) | move it off the lot (`exploit`, `set_flag` moved_line) |

Supporting cards: **the-call-up** (winter, 1927–1930, `requires.flag` affiliated, yearly): the affiliate takes a man (`sell_best` under 27 at a fixed 400) or you break the agreement (`clear_flag` affiliated, `spend` 500). **the-boston-return** (1928 opening, followup of the-boston-offer when kept or matched): he stayed; `set_flag` boston_kept if the promise path was taken and he is still rostered, else the promise breaks on its own.

### 2.3 The sixteen windowed cards (`repeat: never`, weight 40–70)

Local (8): parish-feast (1921–24, the feast asks for the park); grounds-boy (1922–26, a crew boy wants a tryout, `sign`); carvalho-concession (1923–27, the bakery wants the stand, `concession_deal` shape); the-portuguese-paper (1921–25, a column, culture); morelli-protection (1924–28, Prohibition crew offers "protection": `exploit` if taken, `set_flag` morelli); the-flood (1927–30, right-field boards go, repair or demand); the-mill-nine (1921–26, exhibition against the mill's semipros, earn or loyalty); russo-uniforms (1923–28, the uniforms wear out).

National (8): black-sox-verdict (1921, fans doubt a close game, trust/loyalty); radio-craze (1923–25, portable sets in the stands); ruth-sixty (1927, the fence and the lively ball again); lindbergh-day (1927–28, a day at the park); the-scopes-summer (1925–26, a preacher asks for a Sunday); barnstormers (1922–28, a Negro league club asks for a date: the gate's best day; the only touch of the line in M5a, the rest is M5c); florida-bust (1926, a partner's money is gone; the club covers his call: `spend` 400, or lets him out: culture -2); crash-week (1929 block2, fixed-year, the week itself, `demand` -0.10).

### 2.4 Chains
the-strike-year → going-south (1 season). the-affiliate → the-call-up (yearly while `affiliated`). the-boston-offer → the-boston-return (1928) when kept or matched. the-sale → the-line (1 season).

### 2.5 Yearly variants
the-grounds-man, opening-day-band, thin-week and pennant-winter carry four body texts each. A card body may be split by a line containing only `===`; the loader picks `variants[year % n]`. Options and effects do not vary.

## 3. Letters
Every year 1918–1930 has authored prose in three tones (even/warm/cold). New: 1921–1928 and 1930 (27 texts). 1929 stays as written. Tone selection is unchanged (last season's result). One schema addition: a year entry may carry `variants: [{ flag, even, warm, cold }]`, chosen before tone when the flag is set; used by 1927 (`boston_sold` / `boston_matched` / default). Year beats: 1921 short time, subscription holds "this year"; 1922 Ashworth writes as a mill owner, Carvalho supplies the warmth; 1923 "the valley's direction", subscription "reviewed"; 1924 never mentions the quota, Russo's uniforms line returns; 1925 games can be sold "without the inconvenience of a crowd"; 1926 independence is "a sentiment" in the cold tone; 1927 Boston; 1928 Ashworth brief, Carvalho not; 1930 support 0, Carvalho's bread, the shortest letter in the game.

## 4. The ending

### 4.1 When
After the 1930 fall books the walk is the last walk; "Next year" opens the EndPage instead of a 1931 winter. Foreclosure in any year ends the run at once as today, but the foreclosure screen is the EndPage with the finances verdict failed and the retrospective for the years played.

### 4.2 The last walk
Same three places (gate, concession, gus). `toWalk` in 1930 sets flag `last_walk` before readouts are chosen. Twelve new readout entries (`years: [1930, 1930]`, `when` includes `flag: last_walk` plus the band/flag keys), four per place, each revisiting 1918: the family at the gate (the Arruda family if `family_at_gate` was taken, otherwise the mill families; condition from loyalty band and `fed_line` / `moved_line`); the concession spot (Carvalho's if the bakery has it, the chourico vendor if he stayed, Draper's sign if the fence was sold; condition from the deal and the Depression); Gus, 30, still at short, from the trust band. No new readout code.

### 4.3 The three verdicts (pure functions in `src/franchise/verdict.ts`)
- **Baseball**: one of four registers by pennants and winning seasons: ≥3 pennants; 1–2; 0 with ≥6 winning seasons; else. The retrospective names the years.
- **Finances**: four lines, each pass/fail, prose not numbers: no obligations outstanding (no unpaid loans, no open promises); a legal roster (≥ roster minimum, ≤ max); infrastructure ≥ 40 (reads as the grandstand's condition); cash net of debt > 0 (the bar is zero; bots: a careful owner ends $4–15k, a careless one forecloses).
- **The town**: prose from loyalty band, trust band, culture and the leading drift path; names the path: the club that stayed the valley's (roots), the club that reached (ambition), the club that kept its books (money).

### 4.4 The newspaper retrospective
A Providence column over the seasons played, generated from `L.history`, `L.seasons`, the books and flags: pennants by year, the best and worst seasons, three memorable players (highest career overall among men with ≥5 seasons on the Reds; fewer if fewer qualify), cash 1918 vs the final year, debts, and the remembered choices (flags: strike park, affiliated, boston_sold/kept, rally, fed_line/moved_line). Template prose with slots; no numbers beyond years, records and dollars.

### 4.5 Closing screen
Verdicts, the column, then "what carries forward into 1931": cash and debt, the roster's core by name (top five by value), Gus at short, the path, and the grandstand's condition in one line (sound if infrastructure ≥ 40; "the third-base section wants looking at" below 40; "the grandstand should not be used" below 25). This is the only foreshadowing of the Bandbox Collapse. No score, no grade.

### 4.6 Save
`demandMod` and `affiliate` are new franchise fields; save version 7; v6 saves refused like v5.

## 5. Code (Codex) and content (Fable)
Codex: the two effects, `demandMod` in attendance, card body variants in the loader and checker, letter flag variants, `last_walk` flag, `verdict.ts`, EndPage (replaces the stub and the foreclosure headline), save v7, tests. Fable: 26 card files, 27 letter texts, 12 last-walk readouts, retrospective and closing templates; `src/story/content.ts` must be touched for new content files to load under test.

## 6. Tests
- Coverage: fresh league, no flags, every year×slot 1921–1930 has ≥4 eligible cards; every spine card is eligible in its year and slot.
- Effects: `demand` changes attendance for the year only; `sell_best` picks the right man, respects `under`, refuses below the roster minimum.
- Variants: body split and pick by year; checker rejects an empty variant; letter flag variant chosen before tone.
- Verdicts: each baseball register, each finances line pass and fail, each town path, foreclosure mid-run.
- Bots 1918–1930 unchanged as the numbers gate: money-first ends positive, careless forecloses. Tuning rule: ≤3 constants per pass, ≤3 passes, then a finding for Alex.
- Content checked at test time as today.

## 7. Acceptance (the gate)
Codex half: tests green incl. bots, `docs/m5a-build/bots-report.md` every gate passing, build clean. Alex half: a new league, 1918–1930 played or simmed: the spine fires in order; no card repeats word for word; the 1927 letter reads the Boston choice; the last walk revisits the 1918 choices; the three verdicts and the column read true to the run; the grandstand line lands. The human question: does the ending feel earned.
