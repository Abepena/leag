---
phase: 02-wire-the-dynamics
plan: 02
type: execute
wave: 2
depends_on:
  - 02-wire-the-dynamics-01
files_modified:
  - index.html
autonomous: true
requirements:
  - STAND-03
must_haves:
  truths:
    - "Clicking the 'Basketball' chip in #playground swaps the standings card to '12U basketball standings' with Breakers as the .ours-row"
    - "Clicking the 'Soccer' chip swaps to soccer standings with Nets as the .ours-row"
    - "Clicking back to 'Softball' restores Diamonds standings"
    - "Sport chip click also fires existing updateDemo() (does not break theme retint, mockup re-render, or demoState.sport synchronization)"
    - "currentSport state inside the standings IIFE follows the chip — subsequent applyFinalToStandings calls bump the correct sport's ours-row"
  artifacts:
    - path: "index.html"
      provides: "Sport chip click handler additionally calls window.renderRunStandings(sport) so standings swap matches sport-chip selection"
      contains: "window.renderRunStandings(button.dataset.demoSport)"
      contains_at_least: 1
  key_links:
    - from: "[data-demo-sport] click handler at index.html line ~7689"
      to: "window.renderRunStandings"
      via: "appended call inside the existing click listener"
      pattern: "renderRunStandings\\(button\\.dataset\\.demoSport\\)|renderRunStandings\\(demoState\\.sport\\)"
---

<objective>
Wire the existing playground sport chips ([data-demo-sport]) to also re-render the standings card so picking Basketball/Soccer/Softball/Pickleball swaps the standings to that sport with the user's team highlighted.

Purpose: STAND-03. The standings card is data-driven (Plan 03 of Phase 1) but currently only renders softball on boot. Without this wiring, the sport chips have no effect on standings — visitors who pick "Basketball" still see softball standings.

Output: index.html — the existing sport-chip click listener at line ~7689 gets one additional line that calls `window.renderRunStandings(button.dataset.demoSport)`.
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
@.planning/phases/01-lift-the-widget/01-lift-the-widget-03-SUMMARY.md
@./CLAUDE.md

<interfaces>
<!-- The existing sport chip handler — append, do not replace. -->

index.html lines 7689-7694:
```js
document.querySelectorAll("[data-demo-sport]").forEach((button) => {
  button.addEventListener("click", () => {
    demoState.sport = button.dataset.demoSport;
    updateDemo();
  });
});
```

Standings IIFE export (already present from Phase 1 Plan 03):
```js
window.renderRunStandings = render;  // index.html line 9272
```

Sport chip markup (index.html lines ~5999-6003):
```html
<button class="demo-chip is-active" type="button" data-demo-sport="softball">Softball</button>
<button class="demo-chip" type="button" data-demo-sport="basketball">Basketball</button>
<button class="demo-chip" type="button" data-demo-sport="soccer">Soccer</button>
<!-- pickleball may or may not be present as a chip — standings IIFE supports it either way -->
```

The standings IIFE already has data for all 4 sports (softball, basketball, soccer, pickleball) so any value of `data-demo-sport` produces a valid render. If the chip dataset value doesn't match a key, the standings IIFE's render() falls back to softball (`standings[sport] || standings.softball`).
</interfaces>
</context>

<tasks>

<task type="auto">
  <name>Task 1: Append renderRunStandings call to sport-chip click handler</name>
  <read_first>
    - index.html lines 7685-7700 (sport-chip click handler — exact lines to modify)
    - index.html lines 9260-9276 (standings IIFE — confirms window.renderRunStandings is on window and accepts the sport key string)
    - .planning/phases/01-lift-the-widget/01-lift-the-widget-03-SUMMARY.md (confirms public name and accepted sport keys)
  </read_first>
  <files>index.html</files>
  <action>
    Find the existing sport-chip click handler:
    ```js
    document.querySelectorAll("[data-demo-sport]").forEach((button) => {
      button.addEventListener("click", () => {
        demoState.sport = button.dataset.demoSport;
        updateDemo();
      });
    });
    ```

    Add one line at the end of the click callback (AFTER `updateDemo();`):
    ```js
        if (typeof window.renderRunStandings === "function") {
          window.renderRunStandings(button.dataset.demoSport);
        }
    ```

    Final block becomes:
    ```js
    document.querySelectorAll("[data-demo-sport]").forEach((button) => {
      button.addEventListener("click", () => {
        demoState.sport = button.dataset.demoSport;
        updateDemo();
        if (typeof window.renderRunStandings === "function") {
          window.renderRunStandings(button.dataset.demoSport);
        }
      });
    });
    ```

    Reasoning for typeof guard: the click handler at line 7689 lives in the main `<script>` block which parses BEFORE the standings IIFE (line 9191). At parse time the bridge isn't on window yet — but click events only fire after both scripts have executed, so the guard is defensive but cheap.

    Do NOT change the existing demoState.sport assignment. Do NOT change the updateDemo() call. Do NOT add a separate forEach loop or document.querySelectorAll — append inside the existing handler. Do NOT call renderRunStandings(demoState.sport) — the dataset value is already in scope as `button.dataset.demoSport` and is the more direct read (avoids a redundant property lookup on demoState).
  </action>
  <verify>
    <automated>grep -c "window.renderRunStandings(button.dataset.demoSport)" index.html</automated>
  </verify>
  <acceptance_criteria>
    - `grep -c 'window.renderRunStandings(button.dataset.demoSport)' index.html` returns 1
    - `grep -c 'typeof window.renderRunStandings === "function"' index.html` returns 1
    - `grep -c 'data-demo-sport' index.html` returns >= 4 (3 chip buttons + 1 selector — should be unchanged from before)
    - `grep -c 'demoState.sport = button.dataset.demoSport;' index.html` returns 1 (existing line preserved)
    - `grep -c 'updateDemo();' index.html` returns >= 5 (existing call sites preserved — the chip handler still calls updateDemo, plus other call sites in the demo flow)
    - Only ONE forEach over [data-demo-sport]: `grep -c 'querySelectorAll("\[data-demo-sport\]")' index.html` returns 1 (no duplicate listener block introduced).
    - All inline `<script>` blocks still parse: `node -e "const fs=require('fs'); const html=fs.readFileSync('index.html','utf8'); const m=html.match(/<script>([\s\S]*?)<\/script>/g)||[]; m.forEach((s,i)=>{ try { new Function(s.replace(/<\/?script>/g,'')); } catch(e) { console.error('Block', i, 'failed:', e.message); process.exit(1); } }); console.log('OK', m.length);"` → prints `OK <N>`.
  </acceptance_criteria>
  <done>
    Sport chip click → demoState.sport updated → updateDemo() runs → renderRunStandings(sport) re-renders the standings table for that sport. Visitor clicks "Basketball" chip and sees the standings card swap to basketball with Breakers as the highlighted ours-row.
  </done>
</task>

</tasks>

<verification>
End-to-end smoke (manual or via claude-in-chrome):

1. Load `http://localhost:8765/`. Page loads with softball standings (Diamonds highlighted, 5 rows).
2. Scroll to playground top-bar. Click the "Basketball" chip.
3. Expected: standings card title changes to "12U basketball standings", rows show Hammerheads/Breakers/Marlins/Tide/Stingrays with Breakers as the .ours-row (visual highlight + sport-keyed swatch). Other top-bar effects (org name, theme, mockup) still fire as before.
4. Click "Soccer" chip → standings show Nets as ours-row.
5. Click "Softball" chip → standings restore to Diamonds as ours-row, original W/L values (unless Plan 01's bridge fired in between, in which case bumped values persist — that's correct behavior).
6. Click "End game" after Diamonds wins → applyFinalToStandings (from Plan 01) bumps the CURRENT sport's ours-row, not always Diamonds. (E.g. if currently on basketball, Breakers' W/L gets bumped.)
7. Console: zero errors throughout.
</verification>

<success_criteria>
- Sport chip click handler now calls window.renderRunStandings with the chosen sport
- Picking basketball swaps standings to basketball data with Breakers as ours-row
- Picking soccer swaps to soccer data with Nets as ours-row
- Picking softball restores Diamonds standings
- Existing top-bar behavior (org name, theme retint, mockup re-render) still works
- No console errors on chip clicks
- The applyFinalToStandings bridge from Plan 01 now operates on the currently-displayed sport (cross-Plan composition works)
</success_criteria>

<output>
After completion, create `.planning/phases/02-wire-the-dynamics/02-wire-the-dynamics-02-SUMMARY.md` per the template.
</output>
