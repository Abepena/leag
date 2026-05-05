# LEAG Landing Page

## What This Is

Marketing landing page for **leag.app** — multi-tenant SaaS for youth-sports orgs. Single static HTML deployed via GitHub Pages. Brand surface that converts coaches and clubs to platform signups.

## Current State

**Shipped: v1.0 — Step 4 Scoreboard + Standings Swap (2026-05-05)**

Step 4 of the "Try a sample season" accordion now hosts a polished, fully-interactive scoreboard widget + dynamic standings card lifted from `_preview/organize.html`. Hero scoreboard mirrors live state. Standings respond to game outcomes (W/L bump on End game) and to playground sport-chip selection (softball/basketball/soccer/pickleball). Two-column layout on desktop ≥1024px, single column on mobile. Zero console errors, zero duplicate IDs.

## Core Value

Step 4 ("Run the season") is the demo's payoff — visitors who reach it should feel the product. v1.0 raised the perceived product quality of that moment by lifting in production-grade widgets without changing surrounding flow.

## Context

- **Tech**: Single static `index.html` (~9000 lines, all CSS + JS inline), `pricing.html`, plus `brand/` assets. No build step. Deployed via GitHub Pages (`.github/workflows/static.yml`) on push to `main`.
- **Reference source**: `_preview/organize.html` (full-app dashboard mockup) — source of the lifted scoreboard + standings widgets.
- **Existing demo embeds**: index.html now has 4 fully embedded demos in the playground accordion: registrations, teams, publish-schedule, run-the-season (Step 4).
- **Hero scoreboard IDs**: `hero-score-diamonds`, `hero-score-hammerheads`, `hero-line-diamonds-total`, `hero-line-hammerheads-total`, `hero-inning-label`, `hero-score-number` — written by the new state-machine render fn on every state change.
- **Sport selector**: Top-bar in `#playground` lets user pick softball/basketball/soccer; standings card swaps via `window.renderRunStandings(sport)`.
- **Cross-IIFE bridge**: `window.applyFinalToStandings(homeTotal, awayTotal)` exposed by standings IIFE; called by scoreboard endGame fn so W/L bump fires on every Final transition.

## Requirements

### Validated

- ✓ **STEP4-01**: Scoreboard widget HTML lifted into Step 4 — v1.0
- ✓ **STEP4-02**: Standings card HTML lifted (lock-foot stripped, .step4-stack wrapper) — v1.0
- ✓ **STEP4-03**: State machine + handlers replace legacy scoreState/renderScoreDemo/applyScoreAction — v1.0
- ✓ **HERO-01**: Render fn updates 6 hero IDs every state change — v1.0
- ✓ **STAND-01**: Sport-keyed data + `renderRunStandings(sport)`, softball boot — v1.0
- ✓ **STAND-02**: On-final W/L bump + pct recompute + re-render — v1.0
- ✓ **STAND-03**: Sport chips drive standings re-render — v1.0
- ✓ **NOREG-01**: Steps 1-3 of accordion unchanged — v1.0
- ✓ **NOREG-02**: Zero console errors, zero duplicate IDs — v1.0
- ✓ **NOREG-03**: Top-bar controls + theme retint intact — v1.0

### Active

(None — awaiting next milestone scope.)

### Out of Scope

- Adapting the scoreboard widget per sport (innings/balls don't fit basketball/soccer) — scoreboard stays softball regardless of sport pick. Only standings swaps. *Reasoning still valid.*
- Animating standings re-sort with row reorder transition.
- Persisting state across page reloads.
- Other landing page changes (pricing tweaks, copy, hero, marketplace section, etc.) — defer to future milestones.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single GSD project, "just the step 4 swap" scope | Smallest ceremony for focused change | ✓ Good — milestone shipped clean in 2 phases |
| Coarse granularity (2 phases) | Single-file change | ✓ Good — Phase 1 lift, Phase 2 wire |
| Skip research | Source widgets already in `_preview/organize.html` | ✓ Good — no information loss |
| Inherit Opus for all agents | Per user CLAUDE.md, Max plan = sunk cost | ✓ Good — quality consistent |
| Strip lock-foot from standings card | Keep demo clean, drop upsell from workflow | ✓ Good — cleaner demo flow |
| Tie standings to playground sport selector | Cohesive — sport pick reflects everywhere | ✓ Good — sport switch feels alive |
| Bump W/L on "Mark final" | Standings visibly responds to outcome | ✓ Good — End game now has visible payoff |
| Cross-IIFE bridge via window.applyFinalToStandings | Avoid sharing scope; mirror existing window.renderRunStandings pattern | ✓ Good — clean contract |
| Relax bottom-of-7 endGame guard for demo UX | Casual users won't reach 7th inning; payoff was unobservable | ✓ Good — trade-off accepted; revisit if scoreboard surfaces in real-game context |
| 2-column step 4 layout on desktop ≥1024px | Standings card was leaving whitespace on desktop | ✓ Good — better visual density |
| skip_discuss workflow gate ON | Single-file polish doesn't need batch grey-area Q&A | ✓ Good — no workflow friction |

## Constraints

- **Single static HTML deployment**: No build step, no JS framework, no CSS preprocessor. All CSS + JS inline in `index.html`. Constrains plan structure (everything is one file).
- **GitHub Pages deploy on `main` push**: No staging surface; landing changes go straight to production. Verify locally via `python3 -m http.server` or equivalent before commit.
- **Hero scoreboard contract**: 6 hero IDs MUST stay wired. Any future scoreboard work must preserve this binding.
- **No competitor naming in copy**: Per user memory — TeamSnap / GameChanger / etc. forbidden.
- **No raw mailto:**, all email anchors use `class="copy-email" data-email="contact@leag.app"` pattern with click delegate copying to clipboard.

## Evolution

After each phase transition (`/gsd:transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

After each milestone (`/gsd:complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

## Next Milestone Goals

TBD via `/gsd:new-milestone`. Candidates surfaced from launch-week polish work + carried-forward Phase 1 plan-02 SUMMARY notes:
- Standings W/L bump should also fire on natural game-end (currently requires explicit End game click)
- Scoreboard reset semantics — Reset returns to seed state but standings W/L bump persists; might want a "reset everything" button
- Pricing.html polish (untouched in v1.0)
- Marketplace section refinement (untouched in v1.0)
- Mobile sticky controls UX — current sticky position works but could be polished

---
*Last updated: 2026-05-05 after v1.0 milestone*
