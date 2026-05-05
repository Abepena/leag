---
phase: 01-lift-the-widget
plan: 02
type: execute
wave: 2
depends_on:
  - 01-lift-the-widget-01
files_modified:
  - index.html
autonomous: true
requirements:
  - STEP4-03
  - HERO-01

must_haves:
  truths:
    - "Clicking +Diamonds in new scoreboard increments Diamonds widget score AND hero #hero-score-diamonds AND #hero-line-diamonds-total."
    - "Same for +Hammerheads / hero-score-hammerheads / hero-line-hammerheads-total."
    - "Inning advancement updates widget AND hero #hero-inning-label."
    - "Mark final updates hero label to 'Final / Diamonds win' (or Hammerheads/tied) AND #score-status chip."
    - "hero-score-number aria-label rewrites on every score change."
    - "Old scoreState, renderScoreDemo, applyScoreAction, [data-score-action] listener all DELETED — not commented, not layered."
  artifacts:
    - path: "index.html"
      contains: "run-scoreboard"
    - path: "index.html"
      forbidden: "const scoreState ="
    - path: "index.html"
      forbidden: "const renderScoreDemo"
    - path: "index.html"
      forbidden: "const applyScoreAction"
---

<objective>
Port scoreboard state machine from `_preview/organize.html` lines 12450-12749 into index.html. Delete existing scoreState/renderScoreDemo/applyScoreAction/[data-score-action]-listener entirely. New render fn writes hero IDs + #score-status chip.
</objective>

<context>
DELETE region (re-verify exact bounds with grep before editing):
- `index.html` lines ~7936-8093 contains `const scoreState`, `const resetCount`, `const advanceHalfInning`, `const renderScoreDemo`, `const applyScoreAction`, AND `document.querySelectorAll("[data-score-action]").forEach(...)` listener loop.
- `index.html` line ~8105: `renderScoreDemo();` boot call. Replace with `renderRunScoreboard();`.

Re-locate exact bounds:
```
grep -n 'const scoreState' index.html       # start
grep -n 'data-score-action' index.html      # listener
grep -n 'renderScoreDemo()' index.html      # boot
```

Hero IDs the new render writes to:
- `hero-score-diamonds`, `hero-score-hammerheads`, `hero-line-diamonds-total`, `hero-line-hammerheads-total`, `hero-inning-label`, `hero-score-number` (aria-label)
- `score-status` chip in accordion trigger

Team orientation: Diamonds = away, Hammerheads = home. New state machine fields likely `state.away` / `state.home`. Map at hero-write site, not in state machine.

Listener scoping: ALL listeners use `root.querySelectorAll(...)` where `root = document.getElementById("run-scoreboard")` — never `document.querySelectorAll(...)`.
</context>

<tasks>

<task type="auto">
  <name>Task 1: Delete old scoreboard JS and append new state machine IIFE</name>
  <read_first>
    - index.html (lines 7920-8120 to confirm exact bounds; lines 8160-8200 for IIFE pattern)
    - _preview/organize.html (lines 12450-12749 source state machine)
  </read_first>
  <files>index.html</files>
  <action>
**Edit A — Delete old scoreboard code:**
1. `grep -n 'const scoreState' index.html` → start line.
2. `grep -n 'querySelectorAll("\[data-score-action\]")' index.html` → listener loop end.
3. Delete entire range from `const scoreState = {` through closing `});` of listener loop INCLUSIVE.
4. Find `renderScoreDemo();` boot call. Replace with `renderRunScoreboard();`.

**Edit B — Append new IIFE:**
After main script `</script>` close (~line 8158), before registrations IIFE `<script>` (~line 8164), insert:

```html
  <script>
  (function () {
    "use strict";
    const root = document.getElementById("run-scoreboard");
    if (!root) return;

    const $  = (sel) => root.querySelector(sel);
    const $$ = (sel) => Array.from(root.querySelectorAll(sel));

    // STATE — port from _preview/organize.html lines 12450-12749 verbatim.
    const state = { /* state.home, state.away, state.inning, state.half, state.balls, state.strikes, state.outs, state.isFinal */ };

    // HELPERS — port resetCount, advanceHalfInning, endGame, etc.

    // RENDER — port source render. At end, add hero binding:
    function render() {
      // ... port source render body verbatim ...

      // Hero binding (Diamonds = away, Hammerheads = home)
      const heroDia      = document.getElementById("hero-score-diamonds");
      const heroHam      = document.getElementById("hero-score-hammerheads");
      const heroDiaTotal = document.getElementById("hero-line-diamonds-total");
      const heroHamTotal = document.getElementById("hero-line-hammerheads-total");
      const heroInning   = document.getElementById("hero-inning-label");
      const heroAria     = document.getElementById("hero-score-number");
      const chip         = document.getElementById("score-status");

      if (heroDia)      heroDia.textContent      = state.away;
      if (heroHam)      heroHam.textContent      = state.home;
      if (heroDiaTotal) heroDiaTotal.textContent = state.away;
      if (heroHamTotal) heroHamTotal.textContent = state.home;
      if (heroAria)     heroAria.setAttribute("aria-label", `Diamonds ${state.away}, Hammerheads ${state.home}`);

      if (state.isFinal) {
        const dWin = state.away > state.home;
        const tied = state.away === state.home;
        const finalLabel = tied ? "Final / tied" : (dWin ? "Final / Diamonds win" : "Final / Hammerheads win");
        const chipLabel  = tied ? "Final · tied" : (dWin ? "Final · Diamonds win" : "Final · Hammerheads win");
        if (heroInning) heroInning.textContent = finalLabel;
        if (chip)       chip.textContent       = chipLabel;
      } else {
        const halfWord = state.half === "top" ? "Top" : "Bottom";
        if (heroInning) heroInning.textContent = `${halfWord} of ${state.inning}`;
        if (chip)       chip.textContent       = `Live · ${halfWord} ${state.inning}`;
      }
    }

    // LISTENERS — scope to root:
    $$('[data-action]').forEach(btn => btn.addEventListener('click', () => { /* mutate state per data-action */; render(); }));
    $$('[data-count-action]').forEach(btn => btn.addEventListener('click', () => { /* mutate state per data-count-action */; render(); }));
    // + final button + reset button + next-half button per source

    // PUBLIC entry for boot:
    window.renderRunScoreboard = render;

    render();
  })();
  </script>
```

DO NOT add a separate listener loop in main script. ALL listeners inside IIFE.
  </action>
  <acceptance_criteria>
    - `grep -c 'const scoreState' index.html` returns 0
    - `grep -c 'const renderScoreDemo' index.html` returns 0
    - `grep -c 'const applyScoreAction' index.html` returns 0
    - `grep -c 'querySelectorAll("\[data-score-action\]")' index.html` returns 0
    - `grep -c 'document.getElementById("run-scoreboard")' index.html` returns >= 1
    - `grep -c 'window.renderRunScoreboard' index.html` returns >= 1
    - `grep -c 'getElementById("hero-score-diamonds")' index.html` returns >= 1
    - `grep -c 'getElementById("hero-score-hammerheads")' index.html` returns >= 1
    - `grep -c 'getElementById("hero-line-diamonds-total")' index.html` returns >= 1
    - `grep -c 'getElementById("hero-line-hammerheads-total")' index.html` returns >= 1
    - `grep -c 'getElementById("hero-inning-label")' index.html` returns >= 1
    - `grep -c 'getElementById("hero-score-number")' index.html` returns >= 1
    - `grep -c 'getElementById("score-status")' index.html` returns >= 1
    - `grep -c 'renderRunScoreboard()' index.html` returns >= 2 (IIFE call + boot site)
    - All inline `<script>` blocks parse via `node --check` — 0 errors
  </acceptance_criteria>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <name>Task 2: Human verify scoreboard interactivity + hero binding</name>
  <how-to-verify>
    1. Reload `http://localhost:8765/`. DevTools console clean.
    2. Expand Step 4. Confirm: LIVE pill, big Diamonds-vs-Hammerheads scores, linescore, ball/strike/out counters, score +/- buttons, count buttons, next-half/reset/final.
    3. Click +Diamonds twice. Widget score increments. Hero `#hero-score-diamonds` AND `#hero-line-diamonds-total` show new number.
    4. Click +Hammerheads once. Mirror works.
    5. Click Ball/Strike/Out. Widget counter updates. Hero unaffected.
    6. Click Next half-inning. Widget shows new half. Hero `#hero-inning-label` shows e.g. "Bottom of 5".
    7. Score Diamonds higher. Click Mark final. Hero label = "Final / Diamonds win". Trigger chip = "Final · Diamonds win". Hero aria-label updated.
    8. Reset. State returns to initial.
    9. Toggle theme primary swatch. Scoreboard widget retints via `--demo-primary`.
    10. Steps 1, 2, 3 of accordion still work (smoke check).
  </how-to-verify>
  <resume-signal>Type "approved" to continue, or describe failures.</resume-signal>
</task>

</tasks>

<verification>
All grep checks pass. Browser console clean during interactions. `git diff --stat index.html` shows one file changed.
</verification>

<output>
Write `.planning/phases/01-lift-the-widget/01-lift-the-widget-02-SUMMARY.md` per template.
</output>
