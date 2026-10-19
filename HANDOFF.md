# Handoff: Hometown Rafters (2026-10-19)

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
