---
phase: 02-wire-the-dynamics
plan: 03
type: execute
wave: 3
depends_on:
  - 02-wire-the-dynamics-01
  - 02-wire-the-dynamics-02
files_modified:
  - index.html
autonomous: false
requirements:
  - NOREG-01
  - NOREG-02
  - NOREG-03
must_haves:
  truths:
    - "Steps 1, 2, 3 of season-workflow accordion (registrations, teams, publish-schedule) still render and function exactly as before Phase 2"
    - "Page load produces zero console errors"
    - "Scoreboard interactions (+/-, Next half, Prev half, Reset, End game, ball/strike/out) produce zero console errors"
    - "Sport chip clicks produce zero console errors"
    - "No duplicate IDs anywhere in the DOM"
    - "Top-bar controls (org input, sport chips, primary swatches, accent swatches, reset btn) still drive demo state and theme retint correctly"
    - "Scoreboard widget visually re-tints when --demo-primary / --demo-accent CSS vars change via swatch clicks"
  artifacts:
    - path: "index.html"
      provides: "No regressions: all Phase 1 and pre-Phase-1 functionality intact after Plans 01 and 02 of Phase 2"
      contains: "id=\"run-scoreboard\""
      contains_at_least: 1
  key_links:
    - from: "swatch click in #demo-primary-grid / #demo-accent-grid"
      to: "scoreboard CSS rules using var(--demo-primary) / var(--demo-accent)"
      via: "CSS variables on :root, written by wireSwatchRow"
      pattern: "var\\(--demo-(primary|accent)\\)"
---

<objective>
Verify Phase 2 introduced no regressions. Two layered checks:
- Automated grep sweep (script-block parse, ID uniqueness, hook preservation)
- Manual claude-in-chrome smoke (steps 1-3 of accordion, console clean, top-bar drive, theme retint)

Purpose: NOREG-01 / NOREG-02 / NOREG-03. Phase 2 modified the same `index.html` as Phase 1 in a few small spots — verify nothing else moved.

Output: A verification log captured in the checkpoint resume signal. No code changes unless a regression is found, in which case a follow-up gap-closure plan is required (do not patch in this plan).
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/STATE.md
@.planning/REQUIREMENTS.md
@.planning/phases/02-wire-the-dynamics/02-CONTEXT.md
@.planning/phases/02-wire-the-dynamics/02-wire-the-dynamics-01-PLAN.md
@.planning/phases/02-wire-the-dynamics/02-wire-the-dynamics-02-PLAN.md
@.planning/phases/01-lift-the-widget/01-lift-the-widget-01-SUMMARY.md
@.planning/phases/01-lift-the-widget/01-lift-the-widget-02-SUMMARY.md
@.planning/phases/01-lift-the-widget/01-lift-the-widget-03-SUMMARY.md
@./CLAUDE.md
</context>

<tasks>

<task type="auto">
  <name>Task 1: Automated regression-sweep grep + script-block parse</name>
  <read_first>
    - index.html (search-only, no edits)
    - .planning/phases/01-lift-the-widget/01-lift-the-widget-01-SUMMARY.md (lists pre-existing IDs to verify still single-occurrence)
  </read_first>
  <files>index.html (read-only — no edits in this task)</files>
  <action>
    Run the following grep + parse checks. Capture each result. If ANY check fails, do NOT modify code in this plan — record the failure in the Task 2 checkpoint resume signal and surface to user for triage (a separate gap-closure plan addresses the failure).

    Check 1 — All inline `<script>` blocks parse cleanly:
    ```bash
    node -e "const fs=require('fs'); const html=fs.readFileSync('index.html','utf8'); const m=html.match(/<script>([\s\S]*?)<\/script>/g)||[]; m.forEach((s,i)=>{ try { new Function(s.replace(/<\/?script>/g,'')); } catch(e) { console.error('Block', i, 'failed:', e.message); process.exit(1); } }); console.log('OK', m.length);"
    ```

    Check 2 — Required IDs each appear exactly once:
    ```bash
    for id in run-scoreboard run-standings hero-score-diamonds hero-score-hammerheads hero-line-diamonds-total hero-line-hammerheads-total hero-inning-label hero-score-number score-status; do
      count=$(grep -c "id=\"$id\"" index.html)
      echo "$id: $count"
    done
    ```
    Each must print `1`.

    Check 3 — Phase 1 surface still in place:
    ```bash
    grep -c 'window.renderRunScoreboard' index.html   # >= 1
    grep -c 'window.renderRunStandings' index.html    # >= 1
    grep -c 'data-render="run-standings"' index.html  # == 1
    grep -c 'TOTAL_INNINGS' index.html                # >= 2 (declaration + at least one read in nextHalfPreview etc; the constant itself is still referenced after guard removal)
    ```

    Check 4 — Phase 2 wiring in place:
    ```bash
    grep -c 'window.applyFinalToStandings(state.home.total, state.away.total)' index.html  # == 1
    grep -c 'window.renderRunStandings(button.dataset.demoSport)' index.html              # == 1
    grep -c 'function applyFinalToStandings' index.html                                    # == 1
    grep -c 'if (!(state.inning === TOTAL_INNINGS && state.half === "bottom")) return;' index.html  # == 0
    ```

    Check 5 — Other accordion-step demos untouched (their IIFEs / IDs intact):
    ```bash
    grep -c 'id="registrations"' index.html       # == 1
    grep -c 'id="teams"' index.html               # == 1
    grep -c 'id="publish-schedule"' index.html    # == 1
    grep -c 'id="demo-stage"' index.html          # == 1
    ```

    Check 6 — Other globals untouched (legacy or pre-existing surface):
    ```bash
    grep -c 'const demoState' index.html          # == 1
    grep -c 'function updateDemo' index.html      # == 1
    grep -c 'wireSwatchRow' index.html            # >= 3 (declaration + 2 invocations for primary/accent grids)
    ```

    Check 7 — DOM-level duplicate-ID scan (run via claude-in-chrome on the live page):
    ```js
    (function(){var ids={};var dups=[];document.querySelectorAll('[id]').forEach(function(el){var id=el.id;if(ids[id])dups.push(id);else ids[id]=true;});return dups.length?'DUPLICATES: '+dups.join(','):'NO DUPLICATES';})()
    ```
    Expected: `NO DUPLICATES`.

    Record every check's actual output for use in Task 2's resume signal.
  </action>
  <verify>
    <automated>node -e "const fs=require('fs'); const html=fs.readFileSync('index.html','utf8'); const m=html.match(/<script>([\s\S]*?)<\/script>/g)||[]; m.forEach((s,i)=>{ try { new Function(s.replace(/<\/?script>/g,'')); } catch(e) { console.error('Block', i, 'failed:', e.message); process.exit(1); } }); console.log('OK', m.length);"</automated>
  </verify>
  <acceptance_criteria>
    - Check 1 prints `OK <N>` (N = number of inline script blocks; previously confirmed 6 or more)
    - Check 2 prints `1` for every listed ID (no duplicates, no missing)
    - Check 3 produces only positive counts (Phase 1 surface intact)
    - Check 4 produces exactly the listed counts (Phase 2 wiring present, guard removed)
    - Check 5 prints `1` for every listed ID (other demos untouched)
    - Check 6 produces the listed positive counts (legacy surface intact)
    - Check 7 returns `NO DUPLICATES` from the live page (verified via claude-in-chrome)
  </acceptance_criteria>
  <done>
    All 7 automated checks pass. Results captured for Task 2's checkpoint resume signal.
  </done>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <name>Task 2: Manual smoke — accordion steps 1-3, top-bar controls, theme retint, console</name>
  <read_first>
    - .planning/phases/02-wire-the-dynamics/02-wire-the-dynamics-01-PLAN.md (what Plan 01 changed)
    - .planning/phases/02-wire-the-dynamics/02-wire-the-dynamics-02-PLAN.md (what Plan 02 changed)
    - .planning/phases/01-lift-the-widget/01-lift-the-widget-01-SUMMARY.md (pre-existing html-validate noise that's ACCEPTABLE)
    - index.html (only if a regression check fails and the user wants you to grep an anchor)
  </read_first>
  <files>index.html (read-only verification — no edits)</files>
  <what-built>
    Phase 2 added two surgical edits to index.html: (a) standings W/L bump on End game with guard relaxed (Plan 01) and (b) sport-chip → standings re-render (Plan 02). All three Phase 1 widget pieces (markup, scoreboard JS, standings render) remain. Other accordion demos (#registrations, #teams, #publish-schedule), top-bar (#playground), and theming pipeline are untouched by Phase 2 code.
  </what-built>
  <action>
    Pause for human verification — execute the steps documented in `<how-to-verify>` below using claude-in-chrome on http://localhost:8765/. No code edits in this task. If any regression is found, capture the console error / observed mismatch verbatim in the resume signal and surface to user for triage (do NOT patch here — a follow-up `/gsd:plan-phase 02 --gaps` handles regression closure).
  </action>
  <verify>
    <automated>MISSING — manual verification task; automated portion lives in Task 1. Resume signal must include all 6 regression-check PASS markers per `<how-to-verify>`.</automated>
  </verify>
  <how-to-verify>
    Open `http://localhost:8765/` in claude-in-chrome (existing tab id 1647409310 if still alive — otherwise reopen). Open DevTools console BEFORE doing any clicks so you catch boot errors.

    REGRESSION CHECK 1 — Page load (NOREG-02):
    1. Hard reload (Cmd+Shift+R). Console should show zero red errors. Any pre-existing yellow `aria-label` / `role="table"` html-validate-style warnings noted in Phase 1 Plan 01 Issues section are pre-existing and ACCEPTABLE — do not flag.

    REGRESSION CHECK 2 — Steps 1, 2, 3 of accordion (NOREG-01):
    2. Scroll to season-workflow accordion (inside #playground). Expand Step 1 (registrations). Verify the registration form renders with team-color chips and the "Register player" preview area populates as expected. Click around — no console errors.
    3. Expand Step 2 (teams). Verify roster-builder renders with players. Try the demoState.tab switch ("roster" / "stats" / etc). No console errors.
    4. Expand Step 3 (publish-schedule). Verify the schedule preview renders. Try clicking "Generate schedule" or equivalent action. No console errors.

    REGRESSION CHECK 3 — Step 4 happy path (Plan 01 + 02 happy path, also a Phase 1 regression check):
    5. Expand Step 4 (run season). Confirm scoreboard widget visible with controls. Confirm standings card visible with Diamonds highlighted.
    6. Click `+` on Diamonds (home) twice. Verify hero scoreboard at top of page bumps Diamonds 5→6→7 (seed is 5). Verify widget hero shows Diamonds=7.
    7. Click "End game". Verify chip reads "Final · Diamonds win". Verify standings card row for Diamonds: W bumps from 6 → 7, pct recomputes (.750 → .778).
    8. Click "Reset". Verify scoreboard returns to seed (D:5 H:3 Top of 1). Verify standings card stays at the bumped values (W=7).

    REGRESSION CHECK 4 — Sport chip swap (Plan 02 happy path):
    9. Click the "Basketball" chip in playground top-bar. Verify standings card title swaps to "12U basketball standings" with Breakers as the highlighted row. Verify the team mockup also swaps (existing pre-Phase-2 behavior — Step 3 mockup, etc).
    10. Click "Soccer" chip. Standings show Nets as ours-row.
    11. Click "Softball" chip. Standings restore to Diamonds (with the W=7 bump still in effect from step 7 — that's correct, standings mutation persists).

    REGRESSION CHECK 5 — Top-bar controls + theme retint (NOREG-03):
    12. Click a different primary color swatch (e.g. green). Verify --demo-primary CSS var changes site-wide. Specifically inspect: scoreboard live-pill background, scoreboard hero numbers, scoreboard buttons primary fill, hero scoreboard at top of page.
    13. Click a different accent color swatch. Verify --demo-accent CSS var changes — scoreboard button hover states should now use the new accent.
    14. Type a custom org name in the org input. Verify the demo URL preview and any [data-demo-org] elements update.
    15. Click the reset button on top-bar. Verify primary, accent, sport, org all return to defaults. Standings should re-render to softball (because the reset triggers a sport change — verify side effect, not a regression).

    REGRESSION CHECK 6 — Final console scan (NOREG-02):
    16. Console should still show zero red errors after all interactions. Take screenshot or log final console state.

    For each check, mark PASS or FAIL in the resume-signal. If any FAIL, paste the console error / observed mismatch verbatim — do NOT attempt to patch in this plan.
  </how-to-verify>
  <acceptance_criteria>
    - All 6 regression checks marked PASS in resume signal
    - Console clean (zero red errors) at every check
    - No duplicate IDs in DOM (confirmed by Task 1 Check 7)
    - All visible sport chips cycle through standings correctly (softball → basketball → soccer → softball)
    - Theme retint visible on scoreboard widget specifically (var(--demo-primary) / var(--demo-accent) propagates)
    - Steps 1-3 of accordion behave identically to pre-Phase-2 (no demo flow broken)
  </acceptance_criteria>
  <resume-signal>Type "approved — phase 2 verified" if all 6 checks pass. Otherwise paste the failing check number, observed behavior, and any console error verbatim. If a regression is found, do NOT patch in this plan — surface to /gsd:plan-phase 02 --gaps for a follow-up plan.</resume-signal>
  <done>
    All 6 regression checks pass; user types the approval signal. Phase 2 is observably complete: STAND-02 + STAND-03 work in the live page; NOREG-01/02/03 verified.
  </done>
</task>

</tasks>

<verification>
This plan IS the verification step. Task 1 is automated; Task 2 is the live-browser smoke + console scan. No additional phase-level verification needed.
</verification>

<success_criteria>
- All Task 1 grep + parse checks pass (script blocks parse, IDs unique, Phase 1 surface intact, Phase 2 wiring present)
- Task 2 user verification returns "approved — phase 2 verified"
- All 5 phase requirements (STAND-02, STAND-03, NOREG-01, NOREG-02, NOREG-03) demonstrably met
</success_criteria>

<output>
After completion, create `.planning/phases/02-wire-the-dynamics/02-wire-the-dynamics-03-SUMMARY.md` per the template. This SUMMARY closes Phase 2 and marks the milestone complete.
</output>
