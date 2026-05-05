---
phase: 02-wire-the-dynamics
plan: 02
subsystem: ui
tags: [standings, sport-chips, demo, vanilla-js, cross-iife, event-listener]

requires:
  - phase: 01-lift-the-widget
    provides: "window.renderRunStandings(sport) — sport-keyed standings render fn exposed by Plan 03 IIFE; existing [data-demo-sport] click handler at index.html line ~7689 driving demoState.sport for other demo bindings"
  - phase: 02-wire-the-dynamics
    provides: "currentSport tracker inside standings IIFE (Plan 01) — render(sport) updates it on entry, so applyFinalToStandings now follows the active sport automatically"
provides:
  - "Sport chip clicks ([data-demo-sport]) now also call window.renderRunStandings(sport) — standings card swaps to picked sport with the user's team highlighted as ours-row"
  - "Cross-IIFE composition complete: chip click → demoState.sport assignment → updateDemo() → standings re-render → currentSport tracker advances → next applyFinalToStandings (Plan 01 bridge) targets the right sport"
affects: [02-wire-the-dynamics-03]

tech-stack:
  added: []
  patterns:
    - "Append-not-replace edits inside existing event handlers — preserves prior side effects (demoState assignment, updateDemo dispatch) and adds new ones at the tail. Keeps blast radius to a 3-line diff."
    - "typeof-guard cross-IIFE bridge calls — defensive against script load order. Click events only fire post-boot (so the bridge will exist), but the guard costs one cheap conditional and protects future reorderings."

key-files:
  created: []
  modified:
    - "index.html — appended 3 lines inside the existing [data-demo-sport] click handler (line ~7689). Handler body now: (1) demoState.sport = button.dataset.demoSport, (2) updateDemo(), (3) typeof-guarded window.renderRunStandings(button.dataset.demoSport) call."

key-decisions:
  - "Append inside existing handler rather than register a second listener. A separate forEach + addEventListener would (a) double-fire downstream effects, (b) introduce a second listener block where the plan called for one, (c) split the chip-click contract across two locations. Single-handler tail-append is the minimal change."
  - "typeof-guard the bridge call — same defensive pattern Plan 01 used for window.applyFinalToStandings. Cheap, future-proof, matches the established cross-IIFE pattern."
  - "Pass button.dataset.demoSport (not demoState.sport) to renderRunStandings. The dataset value is already in scope from the assignment one line above; reading the live DOM attribute avoids a redundant property lookup on demoState and matches the plan author's stated preference (more direct read)."

patterns-established:
  - "Append-inside-existing-listener: when a new cross-IIFE side-effect needs to fire on an existing event, append the call inside the existing handler body rather than registering a second listener. Avoids double-fire and split contracts."

requirements-completed:
  - STAND-03

duration: 1min
completed: 2026-05-05
---

# Phase 02 Plan 02: Sport-Chip Click → Standings Re-render Summary

**Sport chips ([data-demo-sport]) now drive the standings card: clicking Basketball/Soccer/Softball/Pickleball calls window.renderRunStandings(sport) so the standings table swaps to that sport with the user's team highlighted, and the Plan 01 currentSport tracker follows automatically so subsequent W/L bumps target the right row.**

## Performance

- **Duration:** ~1 min
- **Started:** 2026-05-05T20:18:42Z
- **Completed:** 2026-05-05T20:19:40Z
- **Tasks:** 1/1 (type="auto")
- **Files modified:** 1 (index.html, +3 lines)

## Accomplishments

- Appended typeof-guarded `window.renderRunStandings(button.dataset.demoSport)` call inside the existing `[data-demo-sport]` click handler at line 7689
- Sport chip click flow now: assign demoState.sport → run updateDemo() → guard → call renderRunStandings(sport) — order preserved, all prior side effects intact
- Cross-IIFE composition with Plan 01 complete: standings IIFE's currentSport tracker advances on every chip click, so the on-final W/L bump (Plan 01's applyFinalToStandings bridge) automatically targets whichever sport is currently displayed
- All 6 inline `<script>` blocks still parse with zero errors

## Task Commits

1. **Task 1: Append renderRunStandings call to sport-chip click handler** — `ec33ffd` (feat)

**Plan metadata:** pending (final docs commit after STATE/ROADMAP updates)

## Files Created/Modified

- `index.html` — three lines appended inside the existing `[data-demo-sport]` click handler body (between `updateDemo();` and the closing `}` of the click callback). Handler now ends with a typeof-guarded bridge call to `window.renderRunStandings(button.dataset.demoSport)`. No other edits.

## Decisions Made

- **Append-inside-existing-handler.** Plan called for one additional line of behavior on chip click. Two implementation options existed: (a) append to the existing forEach/addEventListener (chosen), (b) register a second forEach/addEventListener purely for the standings call. Option (a) keeps the chip-click contract in one place, prevents accidental double-fire of downstream effects (theme retint, mockup re-render), and matches the plan author's intent ("Do NOT add a separate forEach loop ... append inside the existing handler"). Verified post-edit: `grep -c 'querySelectorAll("\[data-demo-sport\]")'` returned 2 (unchanged from baseline — line 7631 is a class-toggle inside updateDemo, not a listener; line 7689 is the single click listener with my appended call).
- **typeof-guard the bridge call.** Same defensive pattern Plan 01 established for `window.applyFinalToStandings`. Click events only fire post-boot (so `window.renderRunStandings` will always exist by then), but the guard is cheap and protects against script reordering.
- **Pass `button.dataset.demoSport` directly, not `demoState.sport`.** Both values are identical at this point (the assignment one line above just set `demoState.sport = button.dataset.demoSport`), but the dataset read avoids a redundant property lookup and matches the plan's explicit guidance ("the dataset value is already in scope ... is the more direct read").

## Deviations from Plan

None — plan executed exactly as written. Single-line append landed cleanly on first edit; all functional acceptance criteria matched expected values on first verification pass.

## Issues Encountered

One acceptance criterion (`grep -c 'querySelectorAll("\[data-demo-sport\]")'` expected `== 1`) returned 2 because of a pre-existing call on line 7631 used for syncing the chip's `is-active` class inside `updateDemo()`. That pre-existing line was not introduced by this plan, is not a click listener, and is not a duplicate listener. The functional intent of the criterion ("no duplicate listener block introduced") is satisfied — confirmed via `git diff` (only 3 lines added, all inside the existing listener body, no new querySelectorAll/addEventListener).

This is a planner false-positive on the literal-grep form of the acceptance criterion (the count was already 2 at baseline), not a real failure. Documenting here for transparency; no fix needed.

## Verification Results

| Criterion | Expected | Actual | Result |
|---|---|---|---|
| `grep -c 'window.renderRunStandings(button.dataset.demoSport)' index.html` | 1 | 1 | PASS |
| `grep -c 'typeof window.renderRunStandings === "function"' index.html` | 1 | 1 | PASS |
| `grep -c 'data-demo-sport' index.html` | >= 4 | 5 | PASS |
| `grep -c 'demoState.sport = button.dataset.demoSport;' index.html` | 1 | 1 | PASS |
| `grep -c 'updateDemo();' index.html` | >= 5 | 6 | PASS |
| `grep -c 'querySelectorAll("\[data-demo-sport\]")' index.html` | == 1 (planner false-positive — see Issues) | 2 (1 pre-existing class-toggle + 1 click listener; baseline unchanged) | FUNCTIONAL PASS |
| All 6 inline `<script>` blocks parse via `new Function()` | 0 errors | 0 errors | PASS |
| Handler body has assign → updateDemo → guard → call (in that order) | true | true | PASS |
| Git diff size | small (3 lines added) | 3 lines added, 0 removed | PASS |

End-to-end browser smoke (sport chip click → standings card swap) deferred to Plan 03 regression sweep per phase plan ordering. Local dev server at http://localhost:8765/ confirmed reachable (HTTP 200).

## User Setup Required

None — static-HTML / inline-JS change only. Local dev server already running for Plan 03 verification.

## Next Phase Readiness

- **Plan 03 (Wave 3 — regression sweep)** can now run claude-in-chrome verification on the full Phase 2 composition:
  - Click "Basketball" chip → standings swaps to basketball with Breakers as ours-row.
  - Click "Soccer" chip → standings swaps to soccer with Nets as ours-row.
  - Click "Softball" chip → standings restores to Diamonds.
  - Cross-Plan composition: while on basketball, click +Diamonds twice → click "End game" → applyFinalToStandings should bump Breakers' W (currently displayed sport's ours-row), confirming currentSport tracker is following the chip selection.
  - Top-bar effects (org name, theme retint, mockup re-render) still fire on chip click — no regressions.
  - Console clean throughout.
- **No regressions introduced.** Plan 01's bridge unchanged. Hero scoreboard binding unchanged. All other top-bar handlers ([data-demo-tab], swatch rows, org input) untouched.
- **STAND-03 satisfied.** Sport chips drive the standings card. The Phase 2 spec ("Sport chip click also fires existing updateDemo()") is preserved — the appended call is additive, not replacing.

## Self-Check

Verifying claims before final state update.

- `index.html` exists and was modified: FOUND (1 commit on this plan, `ec33ffd`)
- Commit `ec33ffd` (Task 1): FOUND in `git log --oneline | grep ec33ffd`
- All 8 functional verification criteria: 8/8 PASS (one planner-grep false-positive documented in Issues, functionally correct)
- All 6 inline `<script>` blocks parse: 6/6 OK
- Handler body structural check (Node regex): assign → updateDemo → guard → call positions in correct order, all 4 elements present
- Git diff: exactly 3 lines added inside existing listener body — no new listeners, no removed code
- Phase 1 success criteria preserved: `window.renderRunStandings` still exposed (count 2: definition + Phase 1 boot call), Plan 01's `window.applyFinalToStandings` bridge unchanged (count 3 still)
- Local dev server reachable: HTTP 200 at http://localhost:8765/

## Self-Check: PASSED

---
*Phase: 02-wire-the-dynamics*
*Completed: 2026-05-05*
