# Roadmap

**Project:** LEAG Landing Page — Step 4 Scoreboard + Standings Swap
**Granularity:** coarse
**Total phases:** 2
**Coverage:** 10/10 v1 requirements

## Phases

- [ ] **Phase 1: Lift the widget** — Swap Step 4 body to the polished scoreboard + standings widgets from `_preview/organize.html`, port state machine, preserve hero binding.
- [ ] **Phase 2: Wire the dynamics** — On-final W/L bump, sport-keyed standings swap, regression sweep.

## Phase Details

### Phase 1: Lift the widget

**Status:** Not started
**Goal:** Step 4 of the season-workflow accordion shows the polished scoreboard + standings widgets from `_preview/organize.html`. Scoring works locally. Hero scoreboard at top of page mirrors score live.

**Requirements:** STEP4-01, STEP4-02, STEP4-03, HERO-01, STAND-01

**Success Criteria:**

1. Visitor expands Step 4 and sees the LIVE-pill scoreboard (Diamonds vs Hammerheads with big scores, linescore strip, ball/strike/out controls, +/− buttons, next/prev/final/reset) instead of the old basic scorekeeper.
2. Below the scoreboard, visitor sees the "12U softball standings" card (eyebrow + title + table, no lock-foot) with Diamonds highlighted as the user's row.
3. Clicking +Diamonds twice in Step 4 makes the hero scoreboard's Diamonds score increment from 0 → 2; linescore total and inning label stay in sync.
4. No leftover `data-score-action` buttons or stale `scoreState` / `renderScoreDemo` code paths remain — old scorekeeper is fully replaced, not layered.

**Plans:** 3 plans
- [x] 01-lift-the-widget-01-PLAN.md — Lift step-4 HTML and append scoped CSS for `#run-scoreboard` + `#run-standings`
- [x] 01-lift-the-widget-02-PLAN.md — Port scoreboard state machine + delete old scorekeeper JS + rebind hero IDs
- [x] 01-lift-the-widget-03-PLAN.md — Port sport-keyed standings data + renderRunStandings + boot softball

**UI hint:** yes

---

### Phase 2: Wire the dynamics

**Status:** Not started
**Depends on:** Phase 1
**Goal:** The standings card responds to game outcomes and to the playground sport selector. Nothing else on the page broke.

**Requirements:** STAND-02, STAND-03, NOREG-01, NOREG-02, NOREG-03

**Success Criteria:**

1. After scoring Diamonds higher than Hammerheads and clicking "Mark final," the standings card visibly updates Diamonds' W from N → N+1, recomputes pct, and re-renders the row. Trigger chip reads "Final · Diamonds win" or similar.
2. Picking "Basketball" in the playground sport chips swaps the standings card to "12U basketball standings" with Breakers highlighted as the user's row. Switching back to Softball restores Diamonds standings.
3. Steps 1, 2, and 3 of the season-workflow accordion (registrations, teams roster builder, publish-schedule preview) still render and function exactly as before the swap.
4. Page loads with zero console errors. Interacting with new scoreboard and sport selector produces zero console errors. No duplicate IDs anywhere in the DOM.
5. Top-bar controls (org input, sport chips, primary swatches, accent swatches, reset btn) still drive demo state and theme retint correctly. New scoreboard re-tints with `--demo-primary` / `--demo-accent` swaps.

**Plans:** 3 plans
- [x] 02-wire-the-dynamics-01-PLAN.md - Relax endGame guard + on-final W/L bump bridge (STAND-02)
- [ ] 02-wire-the-dynamics-02-PLAN.md - Wire [data-demo-sport] click to renderRunStandings (STAND-03)
- [ ] 02-wire-the-dynamics-03-PLAN.md - Regression sweep: steps 1-3 intact, console clean, top-bar + theme retint (NOREG-01/02/03)

**UI hint:** yes

---

## Coverage Map

| Requirement | Phase | Notes |
|---|---|---|
| STEP4-01 | Phase 1 | Scoreboard widget HTML lifted |
| STEP4-02 | Phase 1 | Standings card HTML lifted, lock-foot stripped, wrapped in `.step4-stack` |
| STEP4-03 | Phase 1 | State machine ports, replaces existing scoreState/renderScoreDemo/applyScoreAction |
| HERO-01  | Phase 1 | Hero binding moves into new render fn |
| STAND-01 | Phase 1 | Sport-keyed data + `renderRunStandings(sport)` scaffolded; first render = softball |
| STAND-02 | Phase 2 | On-final W/L bump + pct recompute + re-render |
| STAND-03 | Phase 2 | `[data-demo-sport]` click handler triggers re-render |
| NOREG-01 | Phase 2 | Steps 1–3 verified intact |
| NOREG-02 | Phase 2 | Console-clean + dedupe-ID verification |
| NOREG-03 | Phase 2 | Top-bar controls + theme retint verification |

## Progress

| Phase | Status | Started | Completed |
|---|---|---|---|
| 1 — Lift the widget | Not started | — | — |
| 2 — Wire the dynamics | Not started | — | — |
