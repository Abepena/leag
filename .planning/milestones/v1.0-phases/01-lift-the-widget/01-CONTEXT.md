# Phase 1: Lift the widget - Context

**Gathered:** 2026-05-05
**Status:** Ready for planning
**Mode:** Auto-generated (discuss skipped via workflow.skip_discuss)

<domain>
## Phase Boundary

Step 4 of the season-workflow accordion shows the polished scoreboard + standings widgets from `_preview/organize.html`. Scoring works locally. Hero scoreboard at top of page mirrors score live.

Single file modified: `index.html`.

Source widgets (lift verbatim, namespace under `#run-scoreboard` parent ID):
- Scoreboard widget HTML: `_preview/organize.html` lines 6658–6840
- Scoreboard scoped CSS: `_preview/organize.html` lines ~5319–5678
- Scoreboard JS state machine: `_preview/organize.html` lines 12450–12749
- Standings card HTML: `_preview/organize.html` lines 6860–6886 (strip lock-foot)
- Standings sport-keyed data: `_preview/organize.html` lines 9300–9450
- Standings render JS: `_preview/organize.html` lines 9497–9509

Replaces existing in `index.html`:
- Step 4 accordion-panel body
- `scoreState` JS object
- `renderScoreDemo` function
- `applyScoreAction` function
- `[data-score-action]` button-listener loop

Hero binding moves into the new render function — must continue updating IDs `hero-score-diamonds`, `hero-score-hammerheads`, `hero-line-diamonds-total`, `hero-line-hammerheads-total`, `hero-inning-label`, `hero-score-number` (aria-label).

</domain>

<decisions>
## Implementation Decisions

### Widget lift
- Lift wholesale, namespace via parent `#run-scoreboard` ID prefix to avoid collision with hero `.score-hero` styles
- Strip outer `<section class="section">` and `<div class="container">` wrappers — keep `<div class="demo-stage scores-stage">` and inner content
- Strip the standings card's lock-foot (per user lock-foot decision)

### State machine port
- Replace `scoreState` and related functions in main JS — no layering
- New state machine uses its own field names (`state.away`, `state.home`, etc.); orientation: Diamonds = away, Hammerheads = home (matches existing setup and hero binding)
- `renderRunStandings(sport)` exists from Phase 1 and renders default softball; sport-driven swap is Phase 2's STAND-03

### Hero binding inside new render
- Inside the new scoreboard's render function, set `hero-score-diamonds`, `hero-score-hammerheads`, `hero-line-diamonds-total`, `hero-line-hammerheads-total` from state
- Update `hero-inning-label` to mirror inning state ("Top of 5", "Final / Diamonds win")
- Update `hero-score-number` aria-label

### Claude's Discretion
All other implementation choices (variable naming, exact CSS scoping syntax, HTML class naming, internal helper functions). Keep the lifted widget's class names where possible to make CSS port direct.

</decisions>

<code_context>
## Existing Code Insights

### Reusable Assets
- Hero scoreboard IDs already wired: `hero-score-diamonds`, `hero-score-hammerheads`, `hero-line-diamonds-total`, `hero-line-hammerheads-total`, `hero-inning-label`, `hero-score-number`
- CSS variables already on `:root`: `--demo-primary`, `--demo-accent`, `--surface`, `--bg-2`, `--ink`, `--muted`, `--quiet`, `--line`, `--green-hi` — all match what the widget CSS references
- Embedded preview pattern already established in `index.html` for steps 1–3 (registrations, teams, publish-schedule). Each scoped under unique parent ID. CSS appended to main `<style>` block; JS IIFE appended near `</body>`. Same pattern applies here.
- `.acc-demo-host` wrapper class already exists for demos hosted inside accordion panels
- Existing accordion structure: `.accordion-list` > `.accordion-item[data-panel="run"]` > `.accordion-trigger` + `.accordion-panel`. Trigger has `<span class="status-tag" id="score-status">live</span>` chip — keep, update text from new render

### Established Patterns
- Single static `index.html` with all CSS + JS inline (~9000 lines after recent restructure)
- Preview demo embeds use parent ID for CSS scoping (`#registrations`, `#teams`, `#publish-schedule`)
- Old JS that gets replaced is left in place if it gracefully no-ops via null-guard checks (`if (foo)`)
- Hero scoreboard score numbers in DOM use IDs only — written by JS in `renderScoreDemo`

### Integration Points
- Step 4 body inside `<section class="accordion-item" data-panel="run">` — replace the `<div class="accordion-panel">` inner content
- Main `<style>` block (closes around line 4169) — append new scoped CSS just before `</style>`
- Just before `</body>` (around line 8990+) — append new JS IIFE

</code_context>

<specifics>
## Specific Ideas

- Wrap step 4 body in `<div class="step4-stack">` to match `.step3-stack` pattern (top: scoreboard, bottom: standings card)
- Use parent ID `#run-scoreboard` for the scoreboard widget (matches plan)
- Use `data-render="run-standings"` for the standings tbody (renamed from `ov-standings` to avoid future collision)
- Update `#score-status` trigger chip via render: "Live · Top 5", "Final · Diamonds win", "Pre-game" etc.

</specifics>

<deferred>
## Deferred Ideas

- On-final W/L bump → Phase 2 (STAND-02)
- Sport-driven standings swap → Phase 2 (STAND-03)
- Regression sweep for steps 1–3 + console + dup IDs + top-bar → Phase 2 (NOREG-01/02/03)
- Animating standings re-sort with row-reorder transition → out of scope (PROJECT.md)
- Per-sport scoreboard adaptation (basketball/soccer) → out of scope (PROJECT.md)

</deferred>
