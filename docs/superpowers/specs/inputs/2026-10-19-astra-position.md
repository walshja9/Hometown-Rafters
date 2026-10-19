## My position

**Build a baseball game about keeping a particular place alive.** The roster decisions must matter enough that selling a good player hurts before the story explains why. The writing should make that hurt specific to Pawtucket.

I would reject both a narrative game with decorative baseball and an abbreviated OOTP with occasional literary interruptions. The opportunity is the relationship between the two.

The rules and numbers below are my proposed design, not claims that the documents already specify them.

## 1. Challenge the six decisions

**1. Narrative management plus real team management: keep.** Preserve the invisible institutional player. Choices are board resolutions; Gus reacts to teammates, never to an immortal GM. A Moment temporarily puts the player inside one ballplayer’s experience. That distinction is already explicit in [structural-fixes-audit.md:132](<D:/Claude Context/Hometown Rafters/cities/rhode-island-reds/systems/structural-fixes-audit.md:132>).

**2. Hybrid play: keep, with a strict interruption budget.** Two managed games per season, with at most one Moment. Most seasons should not contain a Moment. Being unusual gives them force. Allow simulation of any managed game without sacrificing narrative content.

**3. Tables, period text, simple diamond: keep.** Borrow OOTP’s legibility, not its density. Start with roster, lineup, pitching staff, finances, and standings. Paper styling must never compromise sorting, contrast, keyboard controls, or readable type.

**4. TypeScript and a desktop wrapper: keep, but change “port” to “adapt.”** The Python engine supplies useful matchup mathematics, not a finished simulation. Build browser-first; choose Electron for the first desktop release because its bundled Chromium reduces rendering variation. Accept the larger package. Steam integration comes after the game works. [Electron documentation](https://www.electronjs.org/docs/latest/)

**5. Thirteen seasons: keep as the completed slice.** Explicitly assign 1930 to Era 1 and start the next playable era in 1931. The overlapping headings otherwise invite duplicate simulation.

**6. Thin whole loop: keep, but challenge the production sequence.** First prove one complete season, then three consecutive seasons. Do not author thirteen years before finding out whether anyone understands the consequences.

I also reject an absolute prohibition on winning. The documents say zero championships but expressly permit a local pennant. Let good management earn that pennant; reserve the larger championship mythology for later eras. Never secretly weaken the team to preserve an authored losing record. [era-map-championships.md:130](<D:/Claude Context/Hometown Rafters/cities/rhode-island-reds/systems/era-map-championships.md:130>)

## 2. The playable season

Target **30–40 minutes per season**, approximately seven to nine hours for the campaign, excluding extended roster deliberation.

1. **Winter ledger, 5–8 minutes.** Review last year, renew wages, inspect aging, purchase or trade players, set admission policy, and fund repairs. Ownership supplies a budget directive.
2. **Opening preparations, 3–4 minutes.** Set nine hitters, defensive positions, rotation, and bench. Resolve the first institutional choice.
3. **Season blocks, 6–8 minutes.** Advance through three twenty-game blocks. Pause for two additional choice cards, transactions, and consequential news.
4. **Two story games, 10–14 minutes total.** Manage inning by inning: substitutions, pitcher usage, running aggression, bunting. Plate appearances resolve quickly unless the player requests detail.
5. **Occasional Moment, 1–3 minutes.** Take, swing normally, protect, or bunt; read the count and pitcher tendencies. Decisions influence pitches rather than decorating a predetermined result.
6. **Settlement and ballpark walk, 4 minutes.** Inspect the accounts, then observe three short scenes. Finish with next year’s immediate problem.

Postseason games replace the second managed-game allocation where possible. Additional games remain simulatable.

The strongest existing choice material already joins these layers: ticket assistance, the talented kid needed at the mill, repairs, sponsorship, and trading a founding favorite. [choice-tree.md:49](<D:/Claude Context/Hometown Rafters/cities/rhode-island-reds/systems/choice-tree.md:49>)

## 3. State: separate identity, trust, and survival

The review correctly identifies these as missing executable rules. [REVIEW-2026-10-19.md:96](<D:/Claude Context/Hometown Rafters/REVIEW-2026-10-19.md:96>)

**Path drift:** store Roots, Ambition, and Money scores, each 0–100, initially zero. At each offseason multiply them by 0.85. A substantive decision adds six points to its orientation; a landmark decision adds twelve. Persistent policies count once annually, never whenever a player clicks a control.

Define momentum as the leading score minus the runner-up, divided by 100. Contributions toward another orientation are divided by `1 + momentum`. Established habits therefore take several decisions to reverse. The largest score determines orientation; ties use the most recent substantive choice. No choice becomes unavailable solely because of a path label.

The 1930 Foundation records this accumulated identity. It is not a final menu that overwrites thirteen years, matching [choice-tree.md:69](<D:/Claude Context/Hometown Rafters/cities/rhode-island-reds/systems/choice-tree.md:69>).

**Gus trust:** hidden 0–100, starts at 65. Keeping a meaningful promise adds two; breaking one subtracts five; deliberate exploitation subtracts twelve. Ordinary losses change neither trust nor institutional virtue.

**Institutional culture:** hidden 0–100, starts at 50. Sustained inclusive practices add one per qualifying event; neglect subtracts three; serious betrayal subtracts eight. Positive gains cap at four annually. Damage can happen quickly; repair takes seasons.

**Gus Standard:** compute `0.4 × trust + 0.6 × culture`. Internally use four bands: 0–24, 25–49, 50–74, 75–100. Neither numbers nor band names reach the player. Culture persists beyond Gus’s eventual departure; it is never replaced with a preset determined by path.

**Fan loyalty:** hidden 0–100, starts at 50. Affordable admission adds two annually; exclusionary admission subtracts two; breaking an established community promise subtracts six. Winning or losing seasons add or subtract at most one. Loyalty affects attendance, not batting ability.

**Infrastructure:** 0–100, starts at 65. Full maintenance holds condition; skipping it costs eight annually. A $1,000 repair adds ten. Below 40, unsafe sections close and usable capacity falls.

**League reputation:** 0–100, starts at 50. Honored obligations add two annually; defaults subtract ten. It changes transaction terms by at most ten percent.

Also retain cash, dated liabilities, contracts, roster and game state, community relationships, promises, recurring policies, seen events, and unresolved consequences.

Gus reacts immediately through work, silence, and contextual Portuguese. Crowd behavior changes at season boundaries. Require a three-point crossing beyond a band boundary before reversing a readout, preventing oscillation. This implements the documents’ unequal speeds of damage and recovery. [diegetic-standard-corruption.md:98](<D:/Claude Context/Hometown Rafters/cities/rhode-island-reds/systems/diegetic-standard-corruption.md:98>)

Era 1 must use contemporary signals: Gus arriving early, familiar families returning, vendors extending credit, the newly established Salute. No parking-lot relic, Gus Walk, or Walt yet. Establish the rooster and Salute on screen. The review identifies those missing origins. [REVIEW-2026-10-19.md:64](<D:/Claude Context/Hometown Rafters/REVIEW-2026-10-19.md:64>)

**Money orientation is not yet organized corruption.** Sponsor contracts can restrict admission or concessions, but no mob favors, nephews, investigation clock, or bought umpire calls. Those belong later. Preserve actual obligations when orientation changes; turning toward Roots never cancels a contract. [choice-tree.md:67](<D:/Claude Context/Hometown Rafters/cities/rhode-island-reds/systems/choice-tree.md:67>)

## 4. League, players, and money

Use **six fictional regional clubs**, sixty games each: twelve against every opponent. These are slice opponents, not six newly authored franchise sagas. The top two play a best-of-three local pennant series.

Each club has twenty players: thirteen position players and seven pitchers. Pitchers bat. There is no farm organization, draft administration, or free-agent marketplace.

Generate players from period-specific archetypes: contact shortstop, durable starter, weak-hitting catcher, emerging power hitter. Use 0–99 ratings, retaining 65 as the simulation’s average anchor. Relevant skills are contact, power, discipline, avoidance of strikeouts, speed, fielding, arm, pitching control, strikeout ability, contact suppression, and stamina.

Generate newcomers aged 17–23. Development trends upward through 25, stabilizes through 29, then declines, with seeded variation and earlier speed decline. Start with changes around +1–3 annually for developing skills and −1–3 after peak. Hidden potential bounds growth; playing time matters, but benching someone does not magically destroy talent.

Gus has an authored contact/defense/speed profile and guaranteed career continuity. He cannot be sold or randomly retired in this slice. That is an explicit narrative exception, justified by his fixed presence and later collapse. His batting average remains an outcome, not a forced statistic. [gus-almeida.md:9](<D:/Claude Context/Hometown Rafters/cities/rhode-island-reds/legends/gus-almeida.md:9>)

Contracts renew annually under reserved rights. Players move through trades, cash sales, or purchases from abstract lower-level clubs. Local amateurs receive first contracts. Wage cuts can provoke holdouts and community consequences; “no free agency” must not mean costless coercion.

These are **prototype fictional-dollar values**, not historical salary estimates:

- Opening cash: **$2,000**.
- Annual coalition support through 1928: **$2,000**.
- Payroll: approximately **$4,500**, with individual wages **$125–$750**.
- Travel and operations: **$3,500**.
- Maintenance: **$1,000**.
- Baseline admission: **25¢**, with 700 paid spectators across thirty home games: **$5,250**.
- Net concessions: **5¢ per spectator**, or **$1,050**.
- Ordinary sponsorship: **$700**.

That produces $9,000 income against $9,000 expenses: subsistence, consistent with Ashworth’s expectations. [ownership-lineage.md:28](<D:/Claude Context/Hometown Rafters/cities/rhode-island-reds/ownership/ownership-lineage.md:28>)

Admission at 15¢ raises demand 35%; 40¢ reduces it 25%. Loyalty adjusts demand by one percent per point above or below 50; baseball results contribute at most ±15%. Multiply these factors and enforce usable capacity.

Roots can organize a $500 annual pledge and replace $500 maintenance expense with volunteer work when loyalty reaches 60. Ambition can secure an additional $1,000 performance sponsorship requiring a .450 record. Money can accept a three-year exclusive concession contract paying $1,500 annually **instead of** ordinary concession receipts, with enforceable admission restrictions and community costs. These are arrangements available through decisions, not automatic class bonuses.

Minor-club purchases cost roughly $250–$1,500; established players might sell for $500–$3,000. Buyers must need the player and possess the cash. Use one valuation model with a transaction spread so purchase-and-resale cannot manufacture money.

Loans cap at $3,000, charge five percent, and amortize over three seasons. In 1929 coalition support ends; demand falls ten percent, then twenty-five percent in 1930. The remaining coalition survives without Ashworth’s funding.

If obligations cannot be met after legal borrowing and sales, the attempt ends in a foreclosure newspaper. This is my deliberate survival rule. Every orientation must have a viable route through the downturn; the proposed figures still require balancing.

## 5. One baseball engine

Reuse rating profiles, log5 matchup calculations, and injected randomness. Do not reproduce existing defects: independently calculated strikeout and walk probabilities can exceed total probability one, and the game finishes the bottom half before checking a walk-off. [sim_engine.py:71](<D:/Claude Context/MLB The Show 26 - Reverse Engineer Player Ratings/sim_engine.py:71>) [sim_engine.py:169](<D:/Claude Context/MLB The Show 26 - Reverse Engineer Player Ratings/sim_engine.py:169>)

The current constants explicitly target modern baseball, including 22.5% strikeouts. [sim_engine.py:15](<D:/Claude Context/MLB The Show 26 - Reverse Engineer Player Ratings/sim_engine.py:15>)

Create thirteen annual environment records covering strikeouts, walks, hit batters, home runs, doubles, triples, errors, and runs. Fit against historical aggregates, documenting missing observations. Retrosheet supplies seasonal batting, pitching, fielding, and game data. [Retrosheet downloads](https://www.retrosheet.org/downloads/othercsvs.html)

Do not call 1918–1930 uniformly dead-ball. SABR documents the sharp offensive shift around 1920 and declining strikeouts into the 1920s. Power must become increasingly useful without turning these players into modern hitters. [SABR’s lively-ball history](https://sabr.org/century/1921/lively-ball)

Use one resumable interface:

`stepPA(gameState, action, environment, rngState) → updatedState + baseballEvents`

Log5 supplies matchup targets; a calibrated count-transition model produces pitches consistent with them. Do not independently apply both models and double-count their effects.

Automatic simulation supplies automatic actions. Managed games pause for tactical decisions. Moments request each batting decision. Later arcade inputs supply timing and contact information through an adapter; the common rules still determine legal advancement, outs, and scoring.

Store actual runner identities, count, substitutions, batting order, workload, and fatigue. Support steals, caught stealing, sacrifices, double plays, errors, and correct walk-offs. Runner-only events can end an inning without completing a plate appearance.

Culture changes opportunities, relationships, and atmosphere. **It does not secretly make a virtuous hitter find more gaps.**

## 6. Content and scheduling

Use YAML records with Markdown prose. Each card has an ID, year window, phase, prerequisites, priority, repeat policy, text, options, explicit effects, and follow-up IDs. Effects come from a restricted vocabulary: spend money, create obligation, transfer player, adjust state, set flag. No executable expressions.

Readouts add speaker/location, eligibility, band, and variation text. Moments specify eligible games, actual participants, trigger conditions, batting actions, and success/failure aftermath.

Map the choice tree into approximately thirty-nine decision slots, built from situations and their consequential follow-ups. Ownership supplies mandates; Gus supplies reactions; the era map supplies stakes; the structural audit supplies presentation rules.

Scheduling order is fixed historical scenes, due consequences, then weighted eligible choices. Mandatory consequences replace discretionary slots. Every card has an affordable response, although that response may be painful.

Author six Moment situations across thirteen seasons. An elimination opportunity takes precedence: before a plate appearance, detect a pennant game where losing eliminates the Reds, inning seven or later, with the tying or winning run at bat or on base. Never manufacture that score. If it never occurs, the campaign still concludes.

## 7. Architecture and verification

Keep five boundaries: **baseball rules, franchise rules, content/scheduler, presentation, persistence**. Pure TypeScript rules know nothing about browser widgets or Electron. Run simulation in a worker.

Save complete state, pending cards, in-progress pitches, RNG states, schema version, rules version, and content version. Use transactional browser storage, exportable saves, and rotating desktop backups. Preserve the previous valid save when loading fails.

Borrow the player pool’s identity/import structure, but freeze imported ratings within campaigns. Its re-derivable authoring data is unsuitable as live campaign truth. [player_pool.py:8](<D:/Claude Context/MLB The Show 26 - Reverse Engineer Player Ratings/player_pool.py:8>)

Use separate seeded streams for baseball, development, scheduling, and cosmetic text. Opening a panel must never change tomorrow’s result.

Tests should cover probability bounds, scoring conservation, walk-offs, substitutions, mid-pitch save/reload, and identical results across presentation modes given identical actions. Run large seasonal calibration batches, content reachability checks, and same-seed economic policies. Human playtests must establish whether players can explain *why* the ballpark changed without seeing its hidden state.

## 8. The 1930 ending

The last walk revisits three things established in 1918: a family, a concession place, and Gus’s routine. Their condition supplies the emotional verdict.

The newspaper reports thirteen seasons, pennants, memorable players, cash, and debts. Financial success requires all obligations paid, a legal twenty-player roster, infrastructure at least 40, and cash minus outstanding debt of at least $1,500. Baseball achievement and community legacy remain separate accomplishments.

The Foundation emerges in prose. No morality score appears.

Gus is still thirty and playing shortstop. Do not spend his collapse as a finale. Save the institution’s condition for the next era; its fixed tragedy belongs afterward.

## 9. Ranked risks and cuts

1. **The layers feel disconnected.** Every major choice must alter an actual roster, account, opportunity, or recurring person. Cut unrelated cards first.
2. **One economic strategy dominates.** Test high-price hoarding, repeated sales, cheap labor, and last-year borrowing. Fix incentives before adding events.
3. **Hidden state feels arbitrary.** Keep specific causal reactions and repeated locations. Cut decorative readout variation before clarity.
4. **Baseball expands without limit.** Cut detailed scouting uncertainty, complex injuries, and elaborate trade negotiation. Keep fatigue, defense, running, and substitutions because this era needs them.
5. **Historical texture becomes careless costume.** Apply the review’s period corrections and review representation before commercial release.

If time runs long, reduce two managed games to one, simplify opponents, and defer desktop packaging. Preserve thirteen seasons, meaningful transactions, the park walk, and the shared Moment engine.

## 10. First three builds

1. **The shared baseball seam:** one fixed matchup playable automatically or pitch by pitch, with a diamond, correct scoring, and mid-at-bat save/reload.
2. **One complete 1918:** budget → roster → choices → games → Moment → settlement → park walk → 1919.
3. **A three-season playtest plus a 1930 financial stress scenario:** prove understandable consequences and viable competing strategies before filling out the remaining years.

Nothing was written to disk; I did not consult Claude’s position or run the full test suite. The principal unverified risk is economic balance: these numbers define a testable starting design, not a balanced game.