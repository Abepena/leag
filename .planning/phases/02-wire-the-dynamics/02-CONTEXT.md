# Phase 2: Wire the dynamics - Context

**Gathered:** 2026-05-05
**Status:** Ready for planning
**Mode:** Auto-generated (discuss skipped via workflow.skip_discuss)

<domain>
## Phase Boundary

The standings card responds to game outcomes and to the playground sport selector. Nothing else on the page broke.

Specifically:
- On-final W/L bump: when scoreboard reaches `state.final = true`, the standings card updates the user's team's W (if leading) or L (if trailing or tied) by +1, recomputes pct, re-renders the affected row.
- Sport-driven swap: clicking a sport chip in `#playground` (`[data-demo-sport]`) re-renders standings to that sport's data, with the user's team highlighted (Diamonds for softball, Breakers for basketball, Nets for soccer).
- Regression sweep: steps 1-3 of accordion intact, theme retint works, no console errors, no duplicate IDs, top-bar controls drive demo state.

</domain>

<decisions>
## Implementation Decisions

### On-final W/L bump behavior

- Trigger: scoreboard render fn detects state transition into `state.final = true`. Fires once per transition.
- Logic: if user team total > opponent total → W += 1; else (loss or tie) → L += 1. Tie counts as L for the user's team (matches CLAUDE.md user memory note for Diamonds: pct visibly responds, no special tie column).
- "User team" per sport: softball=Diamonds, basketball=Breakers, soccer=Nets, pickleball=(per data file). The `ours-row` class flag in the standings data identifies the row to bump.
- Re-render: call `renderRunStandings(currentSport)` after mutation so pct + sort order (if any) refresh. Mutation must persist across reset / new game starts within the same page load (state lives on the data object, not in render scope).
- Trigger chip in accordion: reads "Final · Diamonds win" / "Final · Hammerheads win" / "Final · tied" — already produced by Plan 02 render fn at line ~8969 in index.html. No additional work.

### Sport-driven standings swap

- Hook: existing `[data-demo-sport]` click handler in playground top-bar. Search for handler bound to softball/basketball/soccer/pickleball chips.
- Action: handler updates `demoState.sport` (or equivalent) and calls `window.renderRunStandings(sport)`.
- Existing wiring in index.html: top-bar already drives sport for other parts of the demo. Find existing handler, append `renderRunStandings(sport)` call.
- Scoreboard does NOT adapt to sport (scoreboard stays softball-only — this is explicit Out of Scope per PROJECT.md).

### Endgame guard relaxation (carried from Phase 1 Plan 02 SUMMARY)

- Plan 02 ports the source state machine endGame guard: `if (!(state.inning === TOTAL_INNINGS && state.half === "bottom")) return;` (line 9063). This blocks End game from any state but bottom-of-7.
- For demo UX, casual users won't reach bottom-of-7. Solution: remove the guard so End game fires from any state. Let the user "Mark final" whenever they want during the demo flow. Standings W/L bump becomes observable in 3-4 clicks instead of 13+.
- This is Phase 2 work because the W/L bump (the visible payoff) only matters once End game can be triggered casually.

### Regression sweep

- Manual + scripted via claude-in-chrome.
- Console clean throughout: page load, accordion expand, scoreboard interactions, sport switch, theme retint, registrations/teams/publish-schedule demos.
- Duplicate ID scan: `document.querySelectorAll('[id]')` map — no ID appears more than once.
- Top-bar controls: org name input drives `data-demo-org` text replacements, sport chips drive standings + other demo bindings, primary/accent swatches retint via `--demo-primary` / `--demo-accent` CSS vars (verify scoreboard widget retints — buttons + text colors).

### Claude's Discretion

- Plan task ordering, exact selector queries, debug strategy, regression test depth (smoke vs deep).
- Whether to fold endGame guard removal into the same plan as W/L bump or a separate plan. Recommendation: same plan — they're entangled.

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets (from Phase 1)

- `window.renderRunStandings(sport)` — exposed by Plan 03 IIFE. Re-renders `#run-standings` from sport-keyed data.
- Scoreboard render fn (Plan 02 IIFE) — already writes `#score-status` chip with `Final · Diamonds win` / `Final · Hammerheads win` / `Final · tied` labels at line ~8969.
- Hero binding fn — writes 6 hero IDs + chip every render, fires on every state change.

### Established Patterns (from Phase 1)

- IIFE scoping: `root = document.getElementById('run-scoreboard')` / `root = document.getElementById('run-standings')`. All listeners scoped to root.
- Demo state lives on `demoState` object (top-bar driven). Sport key on `demoState.sport`.
- Theme retint via CSS vars `--demo-primary` / `--demo-accent` on `:root` — picker writes to `document.documentElement.style`.

### Integration Points

- Scoreboard render fn — needs hook to fire standings W/L bump on state.final transition. Either:
  - (a) Add a listener inside scoreboard render that detects `state.final` flip from false → true, or
  - (b) Expose `applyFinalToStandings(diamondsTotal, hammerheadsTotal)` from standings IIFE and call it from scoreboard render.
  - Recommend (b) — keeps concerns separated, cleaner cross-IIFE contract.
- Top-bar sport chip handler — find existing `[data-demo-sport]` click delegate, append `renderRunStandings(demoState.sport)` call.

</code_context>

<specifics>
## Specific Ideas

- The CLAUDE.md user memory notes "Diamonds (or sport's 'ours' team) gets +1 W if winning, +1 L if losing." Tie = L confirmed by the source state machine's chip label "Final · tied" (no W bump on tie).
- The scoreboard chip already shows "Final · Diamonds win" — visible in trigger area. No extra work for chip; only standings W/L bump and sport swap.
- Phase 1 Plan 02 SUMMARY flagged the endGame bottom-of-7 guard. Phase 2 will need to relax this so demo users can trigger Final from any state to observe the W/L bump.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope. (Sport-adaptive scoreboard is explicit Out of Scope in PROJECT.md.)

</deferred>
