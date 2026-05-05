# LEAG Landing Page

## What This Is

Marketing landing page for **leag.app** — multi-tenant SaaS for youth-sports orgs. Single static HTML deployed via GitHub Pages. Brand surface that converts coaches and clubs to platform signups.

Active scope of this GSD milestone: **swap the Step 4 of "Try a sample season" accordion to use the polished scoreboard + standings widgets from `_preview/organize.html` overview tab, replacing the current basic scorekeeper.**

## Core Value

Step 4 ("Run the season") is the demo's payoff — visitors who reach it should feel the product. The current scorekeeper looks crude vs. the polished widgets in `_preview/organize.html`. Lifting those widgets in raises perceived product quality without changing the surrounding flow.

Hero scoreboard binding (already wired) must keep working. Standings must respond to "Mark final" by bumping W or L for the user's team.

## Context

- **Tech**: Single static `index.html` (~9000 lines, all CSS + JS inline), `pricing.html`, plus `brand/` assets. No build step. Deployed via GitHub Pages (`.github/workflows/static.yml`) on push to `main`.
- **Reference source**: `_preview/organize.html` is a separate full-app dashboard mockup. Has a richer scoreboard widget at lines 6658–6840 + a sport-keyed standings table at 6860–6886. Drop-in candidates.
- **Existing demo embeds**: index.html has 3 embedded demos already (registrations, teams, publish-schedule) inside the same accordion. Step 4 is the last one to upgrade.
- **Hero scoreboard IDs**: `hero-score-diamonds`, `hero-score-hammerheads`, `hero-line-diamonds-total`, `hero-line-hammerheads-total`, `hero-inning-label`, `hero-score-number` — all live, all written by current `renderScoreDemo`. Must stay wired by the new state machine.
- **Sport selector**: Top-bar in `#playground` lets user pick softball/basketball/soccer. Standings should switch to match.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Replace step 4 body HTML with scoreboard widget lifted from `_preview/organize.html` (lines 6658–6840) + standings card lifted from same (lines 6860–6886, lock-foot stripped)
- [ ] Port scoreboard JS state machine from `_preview/organize.html` (lines 12450–12749) into index.html, replacing existing `scoreState` / `renderScoreDemo` / `applyScoreAction`
- [ ] Hero scoreboard IDs stay wired — new render function updates `hero-score-diamonds`, `hero-score-hammerheads`, `hero-line-diamonds-total`, `hero-line-hammerheads-total`, `hero-inning-label`, `hero-score-number` on every state change
- [ ] Standings table is data-driven and renders from a sport-keyed JS data structure (softball/basketball/soccer/pickleball), via `renderRunStandings(sport)` reading `demoState.sport`
- [ ] On "Mark final" — Diamonds (or sport's "ours" team) gets +1 W if winning, +1 L if losing; pct recomputes; standings re-renders with updated row
- [ ] Sport selector in `#playground` triggers standings re-render via existing `[data-demo-sport]` click handler — picking basketball swaps standings to "12U basketball" with Breakers as `ours-row`
- [ ] No console errors, no duplicate IDs, no broken hero scoreboard sync
- [ ] All other demos (steps 1, 2, 3) still work unchanged after the swap

### Out of Scope

- Adapting the scoreboard widget per sport (innings/balls don't fit basketball/soccer) — scoreboard stays softball regardless of sport pick. Only standings swaps.
- Animating standings re-sort with row reorder transition.
- Persisting state across page reloads.
- Any other landing page changes (pricing tweaks, copy, hero, marketplace section, etc.).

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Use single GSD project, "just the step 4 swap" scope | User chose smallest GSD ceremony for focused change | — Pending |
| Coarse granularity (1–2 phases) | Single-file change, no architectural carving needed | — Pending |
| Skip research | We already have full source for the widgets to lift | — Pending |
| Inherit (Opus) for all agents | Per user CLAUDE.md, Max plan = sunk cost, no downshift | — Pending |
| Strip lock-foot from standings card | Keep demo clean, drop the upsell pitch from the workflow | — Pending |
| Tie standings to playground sport selector | More cohesive — sport pick reflects everywhere it can | — Pending |
| Bump W/L on "Mark final" | Standings visibly responds to game outcome, matches existing renderScoreDemo behavior | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd:transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-05-05 after initialization*
