---
gsd_state_version: 1.0
milestone: v1.0
milestone_name: milestone
status: executing
last_updated: "2026-05-05T19:30:11.070Z"
progress:
  total_phases: 2
  completed_phases: 0
  total_plans: 3
  completed_plans: 1
---

# State

**Project:** LEAG Landing Page — Step 4 Scoreboard + Standings Swap
**Initialized:** 2026-05-05
**Mode:** YOLO
**Granularity:** Coarse
**Total phases:** 2

## Current Position

Phase: 01 (lift-the-widget) — EXECUTING
Plan: 2 of 3 (Plan 01 complete)
**Phase:** 1 of 2 — Lift the widget
**Status:** Executing Phase 01 (Plan 01 ✅, Plan 02 next)
**Next action:** Execute `01-lift-the-widget-02-PLAN.md` (port scoreboard JS state machine + hero rebind)

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

## Phase History

- **01-lift-the-widget Plan 01** (2026-05-05, ~8 min, 2 tasks, commits c59b0cd → 93f5fc1): Step 4 markup swap + scoped CSS for #run-scoreboard / #run-standings. JS still old (broken bindings) — plan 02 will replace.

## Notes

- Local dev server running at `http://localhost:8765/` for verification.
- Browser tab in claude-in-chrome already open for testing (tabId 1647408283).
- Atomic-commit each phase. Use `gsd-tools.cjs commit` for planning-doc commits, regular git commits for code.
