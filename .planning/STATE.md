---
gsd_state_version: 1.0
milestone: v1.0
milestone_name: milestone
status: completed
last_updated: "2026-05-05T23:21:06.263Z"
progress:
  total_phases: 2
  completed_phases: 2
  total_plans: 6
  completed_plans: 6
---

# State

**Project:** LEAG Landing Page — Step 4 Scoreboard + Standings Swap
**Initialized:** 2026-05-05
**Mode:** YOLO
**Granularity:** Coarse
**Total phases:** 2

## Current Position

Phase: 02 (wire-the-dynamics) — EXECUTING
Plan: Not started
**Phase:** 02 of 2 (wire the dynamics)
**Status:** v1.0 milestone complete
**Next action:** Execute `02-wire-the-dynamics-02-PLAN.md` (sport-change handler — Wave 2)

## Accumulated Context

- Implementation plan reference: `~/.claude/plans/ok-help-me-plan-expressive-iverson.md` — fully scoped, decisions locked. Use as primary execution reference.
- Source widgets: `_preview/organize.html` lines 6658–6840 (scoreboard), 6860–6886 (standings), 12450–12749 (scoreboard JS), 9300–9509 (standings data + render).
- Target file: `index.html` only. Single file modified per phase.
- Hero scoreboard binding: IDs already wired (`hero-score-diamonds`, `hero-score-hammerheads`, `hero-line-diamonds-total`, `hero-line-hammerheads-total`, `hero-inning-label`, `hero-score-number`). Must continue updating from new render.
- Decisions made (from PROJECT.md):
  - Strip lock-foot from standings card (clean demo, drop upsell pitch)
  - Tie standings to playground sport selector (more cohesive)
  - On "Mark final" — bump Diamonds W or L based on score outcome
- Decisions made during 01-lift-the-widget Plan 01 execution:
  - Renamed shared keyframes (livedot/bump/cellbump/ticker) to runScoreboard* to avoid collision with hero scoreboard ticker animation
  - Dropped is-locked class and corner-pill from standings card (missing #i-lock SVG sprite, plan stripped lock-foot anyway)
  - Pulled in card/card-head/card-body/.eyebrow/.card-title/.table primitives scoped under #run-standings (those base classes are not present in index.html)
  - Plan acceptance grep added 3 minor CSS rules: #run-scoreboard .scores-board (container), #run-standings .ours-row (simplified selector), .step4-stack mobile media query
- Decisions made during 02-wire-the-dynamics Plan 01 execution:
  - Cross-IIFE bridge pattern: standings IIFE exposes `window.applyFinalToStandings`; scoreboard `endGame` calls it (typeof-guarded) after `state.final` flip. Keeps each IIFE root-scoped, avoids tight coupling.
  - Relaxed both `endGame` body guard AND Final-button disabled rule together (relaxing only one would leave the relaxation invisible — button stuck disabled or body returning early).
  - Tie counts as L for the user team (matches PROJECT.md and 02-CONTEXT.md spec — `if (homeTotal > awayTotal) W += 1; else L += 1;`).
  - `currentSport` tracker added inside standings IIFE (defaults to "softball", set on every render). Decouples bridge from `demoState.sport` — Phase 2 Plan 02's sport-change handler will call `renderRunStandings(sport)`, currentSport tracks automatically.
  - pct format `.toFixed(3).replace(/^0/, "")` to match seed format ("0.875" → ".875").

## Phase History

- **01-lift-the-widget Plan 01** (2026-05-05, ~8 min, 2 tasks, commits c59b0cd → 93f5fc1): Step 4 markup swap + scoped CSS for #run-scoreboard / #run-standings. JS still old (broken bindings) — plan 02 will replace.
- **02-wire-the-dynamics Plan 01** (2026-05-05, ~2 min, 3 tasks, commits e921461 → c732706): Relaxed endGame guard (button + body), exposed `window.applyFinalToStandings` bridge from standings IIFE, wired scoreboard endGame to fire bridge once per final transition. STAND-02 complete. Standings card now visibly bumps Diamonds W or L on End game.

## Notes

- Local dev server running at `http://localhost:8765/` for verification.
- Browser tab in claude-in-chrome already open for testing (tabId 1647408283).
- Atomic-commit each phase. Use `gsd-tools.cjs commit` for planning-doc commits, regular git commits for code.
