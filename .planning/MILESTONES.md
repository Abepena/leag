# Milestones

## v1.0 Step 4 Scoreboard + Standings Swap (Shipped: 2026-05-05)

**Phases completed:** 2 phases, 6 plans, 10 tasks

**Key accomplishments:**

- Step 4 accordion body now hosts the lifted scoreboard (ticker + score-hero + linescore + recap + controls) and the lock-foot-stripped standings card, both scoped under #run-scoreboard / #run-standings, ready for plan 02 to bind JS.
- Scoreboard widget is fully interactive — all controls live, hero scoreboard mirrors every state change, console clean.
- Step 4 standings card renders a 5-row softball table on page load — Diamonds highlighted as the user's row via .ours-row, sport-keyed data structure ready for Phase 2 to swap on sport-change.
- Step 4 standings card now visibly responds to "End game": Diamonds W or L bumps and pct recomputes, with the bottom-of-7 guard relaxed so the bump is reachable in 3-4 demo clicks instead of 13+.
- Sport chips ([data-demo-sport]) now drive the standings card: clicking Basketball/Soccer/Softball/Pickleball calls window.renderRunStandings(sport) so the standings table swaps to that sport with the user's team highlighted, and the Plan 01 currentSport tracker follows automatically so subsequent W/L bumps target the right row.
- Static regression sweep on index.html confirms Phase 2's two surgical edits (Plan 01's endGame guard relaxation + applyFinalToStandings bridge, Plan 02's sport-chip → standings re-render) landed cleanly with no collateral damage: all 6 inline script blocks parse, all required IDs unique, Phase 1 surface intact, other accordion demos and legacy globals untouched. Live-page browser smoke (Task 2) deferred to orchestrator inline verification via claude-in-chrome.

---
