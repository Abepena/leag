---
phase: 02-wire-the-dynamics
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - index.html
autonomous: true
requirements:
  - STAND-02
must_haves:
  truths:
    - "User can click 'End game' from any state (not just bottom-of-7) and the scoreboard goes to FINAL"
    - "When Diamonds total > Hammerheads total at final, standings card updates Diamonds W (N -> N+1) and pct recomputes"
    - "When Diamonds total <= Hammerheads total at final (loss or tie), standings card updates Diamonds L (N -> N+1) and pct recomputes"
    - "Standings re-render is fired from inside the scoreboard endGame transition (not on every render)"
    - "Reset followed by another End game can bump again (mutation persists across resets within the page session, fires once per final transition)"
  artifacts:
    - path: "index.html"
      provides: "endGame guard relaxed; window.applyFinalToStandings(homeTotal, awayTotal) bridge fn exposed by standings IIFE; scoreboard endGame calls it after state.final flip"
      contains: "applyFinalToStandings"
      contains_at_least: 3
      forbidden:
        - "if (!(state.inning === TOTAL_INNINGS && state.half === \"bottom\")) return;"
  key_links:
    - from: "scoreboard IIFE endGame()"
      to: "window.applyFinalToStandings"
      via: "function call after state.final = true; render();"
      pattern: "applyFinalToStandings\\(state\\.home\\.total, ?state\\.away\\.total\\)"
    - from: "standings IIFE"
      to: "window.applyFinalToStandings"
      via: "expose mutator on window so cross-IIFE caller (scoreboard) can fire it"
      pattern: "window\\.applyFinalToStandings"
---

<objective>
Make "Mark final" observable in the demo by (a) relaxing the bottom-of-7 guard so casual users can hit it from any state, and (b) wiring a cross-IIFE hook that bumps the user's team W/L row in standings on every final transition.

Purpose: STAND-02 is the demo's payoff. Without the guard relaxation, the bump is unreachable in the casual flow. Without the bridge, scoring more runs has no visible effect on standings.

Output: index.html — endGame guard removed (or relaxed); standings IIFE exposes `window.applyFinalToStandings(homeTotal, awayTotal)`; scoreboard IIFE endGame() calls that bridge after flipping state.final.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@.planning/STATE.md
@.planning/ROADMAP.md
@.planning/REQUIREMENTS.md
@.planning/phases/02-wire-the-dynamics/02-CONTEXT.md
@.planning/phases/01-lift-the-widget/01-lift-the-widget-02-SUMMARY.md
@.planning/phases/01-lift-the-widget/01-lift-the-widget-03-SUMMARY.md
@./CLAUDE.md

<interfaces>
<!-- Existing surface from Phase 1 (do not redesign). Read these regions of index.html before editing. -->

Scoreboard endGame (index.html lines 9061-9068):
```js
function endGame() {
  if (state.final) return;
  if (!(state.inning === TOTAL_INNINGS && state.half === "bottom")) return;  // GUARD TO RELAX
  state.final = true;
  render();
  const winner = state.home.total > state.away.total ? TEAMS.home : TEAMS.away;
  announce(`Final. ${winner.name} win ${...}.`);
}
```

Scoreboard render disabled-state check (index.html line 8886):
```js
if (elFinalBtn)  elFinalBtn.disabled = !(state.inning === TOTAL_INNINGS && state.half === "bottom");
```
This MUST also be relaxed so the button is clickable outside bottom-of-7. Otherwise relaxing the guard is invisible — the button stays disabled.

Scoreboard final chip writes (index.html ~lines 8965-8971):
```js
if (state.final) {
  const dWin = state.home.total > state.away.total;   // Diamonds wins if home wins
  const tied = state.home.total === state.away.total;
  const finalLabel = tied ? "Final / tied" : (dWin ? "Final / Diamonds win" : "Final / Hammerheads win");
  const chipLabel  = tied ? "Final · tied" : (dWin ? "Final · Diamonds win" : "Final · Hammerheads win");
  ...
}
```
This already writes "Final · Diamonds win" / "Final · Hammerheads win" / "Final · tied" — Phase 2 does NOT modify the chip.

Standings IIFE (index.html lines ~9191-9276):
- `standings` data object lives in IIFE scope (NOT on window). Phase 2 must mutate via the new bridge fn, not directly.
- `render(sport)` is the local render fn; `window.renderRunStandings = render;` is the public name.
- `renderRunStandings("softball")` boot call at line 9274 — leave untouched.
- Each row shape: `{ name, w, l, pct }`. The `ours` field on each sport object identifies the user's team by name.

Diamonds = home team (state.home), Hammerheads = away (state.away). Source of truth: comments at scoreboard hero binding (line 8950) `Widget: Diamonds=home, Hammerheads=away.`
</interfaces>
</context>

<tasks>

<task type="auto">
  <name>Task 1: Relax endGame guard + Final-button disabled rule</name>
  <read_first>
    - index.html lines 9055-9075 (endGame fn)
    - index.html lines 8880-8895 (Final-button disabled check inside render)
    - .planning/phases/01-lift-the-widget/01-lift-the-widget-02-SUMMARY.md (notes the guard as Phase 2 work)
  </read_first>
  <files>index.html</files>
  <action>
    Two precise edits inside the scoreboard IIFE.

    EDIT A — endGame guard (line ~9063):
    Delete the line: `if (!(state.inning === TOTAL_INNINGS && state.half === "bottom")) return;`
    Keep the existing `if (state.final) return;` line above it (idempotency — endGame still fires once per final transition).
    Result: endGame() is now reachable from any (inning, half) where state.final is false.

    EDIT B — Final-button disabled rule (line ~8886):
    Change: `if (elFinalBtn)  elFinalBtn.disabled = !(state.inning === TOTAL_INNINGS && state.half === "bottom");`
    To:     `if (elFinalBtn)  elFinalBtn.disabled = state.final;`
    Result: Final button is clickable from any state where the game isn't already final.

    Do NOT change endGame's body other than the guard line. Do NOT change the chip-writing code at lines ~8965-8971. Do NOT touch the hero binding code. Do NOT widen the diff.
  </action>
  <verify>
    <automated>grep -n "state.inning === TOTAL_INNINGS && state.half === \"bottom\"" index.html | wc -l | tr -d ' '</automated>
  </verify>
  <acceptance_criteria>
    - `grep -c 'if (!(state.inning === TOTAL_INNINGS && state.half === "bottom")) return;' index.html` returns 0 (guard removed)
    - `grep -c 'elFinalBtn.disabled = state.final;' index.html` returns 1 (new disabled rule)
    - `grep -c 'elFinalBtn.disabled = !(state.inning === TOTAL_INNINGS && state.half === "bottom")' index.html` returns 0 (old disabled rule removed)
    - `grep -c 'function endGame()' index.html` returns 1 (function still exists, just different body)
    - `grep -c 'if (state.final) return;' index.html` returns >= 6 (other state.final guards across nextHalf/prevHalf/ball/strike/addOut/etc preserved — no collateral damage)
    - `grep -n "state.inning === TOTAL_INNINGS && state.half === \"bottom\"" index.html` may still match in `nextHalfPreview` or other read-only checks; that's fine. The only forbidden form is the endGame `return;` guard and the Final-button disabled check, both replaced above.
  </acceptance_criteria>
  <done>
    Final button enabled from any non-final state; endGame() body no longer rejects mid-game calls.
  </done>
</task>

<task type="auto">
  <name>Task 2: Expose window.applyFinalToStandings bridge from standings IIFE</name>
  <read_first>
    - index.html lines 9191-9276 (full standings IIFE — data shape, render fn, window export)
    - .planning/phases/01-lift-the-widget/01-lift-the-widget-03-SUMMARY.md (data structure decisions)
  </read_first>
  <files>index.html</files>
  <action>
    Inside the standings IIFE (between the existing `window.renderRunStandings = render;` line and the boot call `renderRunStandings("softball");`), add a bridge function that mutates the current sport's ours-row W or L by +1 and re-renders.

    Add a `currentSport` variable to track which sport is currently rendered. Update the local `render(sport)` fn to set `currentSport = sport;` at the top so the bridge knows which sport to mutate. Then add the bridge.

    Insert this code AFTER the existing render() fn definition and BEFORE `window.renderRunStandings = render;`:

    ```js
        let currentSport = "softball";

        function applyFinalToStandings(homeTotal, awayTotal) {
          // Diamonds = home in scoreboard. The user's team in standings = data.ours.
          // For sports other than softball the scoreboard doesn't run (it's softball-only),
          // but we still treat home-total as the user's-team total for symmetry.
          const data = standings[currentSport] || standings.softball;
          const oursRow = data.rows.find(function (r) { return r.name === data.ours; });
          if (!oursRow) return;
          if (homeTotal > awayTotal) {
            oursRow.w += 1;
          } else {
            oursRow.l += 1;
          }
          const games = oursRow.w + oursRow.l;
          const pct = games > 0 ? (oursRow.w / games) : 0;
          // Format like ".875" — no leading zero, three decimals.
          oursRow.pct = pct.toFixed(3).replace(/^0/, "");
          render(currentSport);
        }

        window.applyFinalToStandings = applyFinalToStandings;
    ```

    Then change the local `render(sport)` to set `currentSport`:
    Find: `function render(sport) {`
    Add as first line of body: `currentSport = sport;`

    Do NOT change the standings data shape. Do NOT touch rowFor() or the boot call. Do NOT add this code outside the IIFE — it must close over the standings data object.
  </action>
  <verify>
    <automated>grep -c "applyFinalToStandings" index.html</automated>
  </verify>
  <acceptance_criteria>
    - `grep -c 'applyFinalToStandings' index.html` returns >= 3 (function declaration + window assignment + at least one call site reserved for Task 3, but task 3 hasn't run yet so >= 3 is correct: declaration + window export + comment-free)
    - `grep -c 'window.applyFinalToStandings' index.html` returns 1
    - `grep -c 'function applyFinalToStandings' index.html` returns 1
    - `grep -c 'currentSport' index.html` returns >= 2 (declaration + at least one read in render and bridge body)
    - `grep -c 'oursRow.pct = pct.toFixed(3).replace' index.html` returns 1 (pct recompute uses .toFixed(3) format)
    - `grep -c 'window.renderRunStandings = render;' index.html` returns 1 (existing export untouched)
    - `grep -c 'renderRunStandings("softball")' index.html` returns 1 (boot call untouched)
    - Open `index.html` in `node --check` equivalent: file parses with no SyntaxError. Run `node -e "const fs=require('fs'); const html=fs.readFileSync('index.html','utf8'); const m=html.match(/<script>([\s\S]*?)<\/script>/g)||[]; m.forEach((s,i)=>{ try { new Function(s.replace(/<\/?script>/g,'')); } catch(e) { console.error('Block', i, 'failed:', e.message); process.exit(1); } }); console.log('OK', m.length);"` → prints `OK <N>`.
  </acceptance_criteria>
  <done>
    Bridge fn declared, exposed on window, mutates ours-row W or L, recomputes pct via .toFixed(3) with leading-zero strip, calls render(currentSport).
  </done>
</task>

<task type="auto">
  <name>Task 3: Fire applyFinalToStandings from scoreboard endGame</name>
  <read_first>
    - index.html lines 9061-9068 (endGame fn — already edited in Task 1)
    - index.html lines 8950-8977 (hero binding — confirms Diamonds=home, Hammerheads=away)
  </read_first>
  <files>index.html</files>
  <action>
    Inside the scoreboard IIFE's `endGame()` fn (the one edited in Task 1), call the bridge AFTER state.final is set and AFTER `render();` runs.

    Find:
    ```js
    function endGame() {
      if (state.final) return;
      state.final = true;
      render();
      const winner = state.home.total > state.away.total ? TEAMS.home : TEAMS.away;
      announce(`Final. ${winner.name} win ${Math.max(state.home.total, state.away.total)} to ${Math.min(state.home.total, state.away.total)}.`);
    }
    ```

    Insert one line after `render();` and before `const winner = ...`:
    ```js
      if (typeof window.applyFinalToStandings === "function") {
        window.applyFinalToStandings(state.home.total, state.away.total);
      }
    ```

    Reasoning for typeof guard: scoreboard IIFE runs before standings IIFE (script-tag order). On initial parse the bridge may not be on window yet, but endGame is only triggered by user click — by then standings IIFE has booted. The guard is defensive in case script load order changes.

    Do NOT call applyFinalToStandings from any other location. Do NOT call it from render() — that would fire on every state change. Only the endGame transition.
  </action>
  <verify>
    <automated>grep -c "window\.applyFinalToStandings(state\.home\.total" index.html</automated>
  </verify>
  <acceptance_criteria>
    - `grep -c 'window.applyFinalToStandings(state.home.total, state.away.total)' index.html` returns 1
    - `grep -c 'typeof window.applyFinalToStandings === "function"' index.html` returns 1 (typeof guard present)
    - `grep -c 'applyFinalToStandings' index.html` returns >= 4 (Task 2 added 3, Task 3 adds 1 call site = total >= 4. If grep counts call site twice — once for typeof line, once for invocation — then >= 5 is acceptable.)
    - The call site is inside endGame(): `awk '/function endGame\(\)/,/^    }/' index.html | grep -c 'applyFinalToStandings'` returns 2 (typeof guard + invocation, both inside endGame).
    - The call site is NOT inside render() or renderCount() or hero binding: `awk '/function render\(\)/,/^    }/' index.html | grep -c 'applyFinalToStandings'` returns 0.
    - All inline `<script>` blocks still parse: `node -e "const fs=require('fs'); const html=fs.readFileSync('index.html','utf8'); const m=html.match(/<script>([\s\S]*?)<\/script>/g)||[]; m.forEach((s,i)=>{ try { new Function(s.replace(/<\/?script>/g,'')); } catch(e) { console.error('Block', i, 'failed:', e.message); process.exit(1); } }); console.log('OK', m.length);"` → prints `OK <N>`.
  </acceptance_criteria>
  <done>
    Scoreboard endGame() calls window.applyFinalToStandings exactly once per final transition. Demo flow: open Step 4, click +Diamonds twice, click "End game" → standings card visibly bumps Diamonds W from 6 to 7 and pct recomputes from .750 to a higher value.
  </done>
</task>

</tasks>

<verification>
End-to-end smoke (manual or via claude-in-chrome):

1. Load `http://localhost:8765/`. Open Step 4 of the season-workflow accordion.
2. Confirm "End game" button is enabled at top of 1st (was disabled before Task 1).
3. Click `+Diamonds` (the home plus button) 2x. Diamonds=2, Hammerheads=0.
4. Click "End game".
5. Expected: chip shows "Final · Diamonds win". Standings card row for Diamonds: W goes from 6 → 7, L stays 2, pct recomputes from `.750` to `.778`.
6. Click "Reset". Score returns to seed (D:5, H:3 from initial state, NOT 0:0 — that's the source state machine seed). Standings card stays at the bumped values (W=7, L=2 — mutation persists across reset within page session).
7. Click "End game" again. Diamonds total (5) > Hammerheads total (3), so Diamonds W bumps again: 7 → 8.
8. Console: zero errors throughout.
</verification>

<success_criteria>
- endGame guard removed; Final button enabled from any non-final state
- window.applyFinalToStandings exposed; mutates ours-row W or L by +1; recomputes pct
- Scoreboard endGame calls bridge exactly once per final transition
- Standings card visibly responds to "End game" with Diamonds total > Hammerheads total → +1 W
- Standings card visibly responds to "End game" with Diamonds total <= Hammerheads total → +1 L (tie counts as L)
- All inline `<script>` blocks parse cleanly
- Hero scoreboard binding from Phase 1 still updates on every state change (no regression)
</success_criteria>

<output>
After completion, create `.planning/phases/02-wire-the-dynamics/02-wire-the-dynamics-01-SUMMARY.md` per the template.
</output>
