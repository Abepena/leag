---
phase: 01-lift-the-widget
plan: 03
type: execute
wave: 3
depends_on:
  - 01-lift-the-widget-02
files_modified:
  - index.html
autonomous: true
requirements:
  - STAND-01

must_haves:
  truths:
    - "Standings card under #run-standings renders multiple rows of softball standings on initial page load."
    - "Diamonds highlighted as user's row via .ours-row + .ours classes."
    - "Rows show team name + chip swatch, W, L, (T if used), pct."
    - "window.renderRunStandings(sport) function exposed globally; accepts 'softball' / 'basketball' / 'soccer' / 'pickleball'."
    - "Sport-keyed data structure mirrors organize.html lines 9300-9450."
    - "renderRunStandings('softball') called automatically on page load."
  artifacts:
    - path: "index.html"
      contains: "renderRunStandings"
    - path: "index.html"
      contains: "softball"
---

<objective>
Port sport-keyed standings data + renderRunStandings function from `_preview/organize.html` into index.html. Render softball on initial load, Diamonds as ours-row. Sport-driven swap and on-final W/L bump are OUT OF SCOPE for Phase 1 (Phase 2 STAND-02/03).
</objective>

<context>
Source in `_preview/organize.html`:
- Sport-keyed standings data: lines 9300-9450
- Render function: lines 9497-9509 (likely `renderOvStandings(sport)`)
- Helper: `teamColorMap` if used

Target tbody (created in Plan 01): `<tbody data-render="run-standings"></tbody>` inside `#run-standings`.

Renaming:
- `renderOvStandings` → `renderRunStandings`
- `[data-render="ov-standings"]` → `[data-render="run-standings"]` (already done in Plan 01)

Sample naming canon (per MEMORY.md):
- Softball: Diamonds (ours)
- Basketball: Breakers (ours)
- Soccer: Nets (ours)
- Universal opponents: Marlins, Hammerheads, Comets, Tide, Stingrays
- Org: Bayside County Sports

Hammerheads must appear in softball standings so it agrees with the scoreboard above.
</context>

<tasks>

<task type="auto">
  <name>Task 1: Port standings data + renderRunStandings + boot softball</name>
  <read_first>
    - index.html (the new IIFE block from Plan 02; embedded-IIFE pattern at lines 8164+)
    - _preview/organize.html (lines 9300-9450 data, 9497-9509 render, nearby teamColorMap if present)
  </read_first>
  <files>index.html</files>
  <action>
Add a new `<script>` block AFTER the scoreboard IIFE (added in Plan 02), before the registrations IIFE:

```html
  <script>
  (function () {
    "use strict";
    const root = document.getElementById("run-standings");
    if (!root) return;
    const tbody = root.querySelector('[data-render="run-standings"]');
    if (!tbody) return;

    // DATA — port from _preview/organize.html lines 9300-9450, adjusted to canon
    const standings = {
      softball: {
        ours: "Diamonds",
        teams: [
          // { team, w, l, t (optional), pct, swatch }
          // Diamonds + opponents from canon (Marlins/Hammerheads/Comets/Tide/Stingrays)
        ],
      },
      basketball: { ours: "Breakers", teams: [/* Breakers + opponents */] },
      soccer:     { ours: "Nets",     teams: [/* Nets + opponents */] },
      pickleball: { ours: /* per source */, teams: [/* ... */] },
    };

    function rowFor(team, isOurs) {
      // Build <tr class="ours-row" if isOurs>...</tr>
      // Cells: team chip (.team-chip > .team-swatch.<class> + name), W, L, (T), pct
      // Use <span class="ours">name</span> wrapper if isOurs
      return /* template literal */;
    }

    function render(sport) {
      const data = standings[sport] || standings.softball;
      tbody.innerHTML = data.teams.map(t => rowFor(t, t.team === data.ours)).join("");
    }

    window.renderRunStandings = render;
    render("softball");
  })();
  </script>
```

Notes:
1. Read `_preview/organize.html` lines 9300-9450 to capture exact data shape (W/L only or W/L/T).
2. Read lines 9497-9509 for render — mirror its row template precisely so CSS from Plan 01 Task 2 matches the markup.
3. If source uses different team names, substitute with canon. Softball MUST have Diamonds as ours + Hammerheads as opponent.
4. Do NOT add sport-change listener — Phase 2 STAND-03.
5. Do NOT add on-final hook — Phase 2 STAND-02.
  </action>
  <acceptance_criteria>
    - `grep -c 'window.renderRunStandings' index.html` returns >= 1
    - `grep -cE "renderRunStandings\\(['\"]softball['\"]\\)" index.html` returns >= 1 (boot call)
    - `grep -c 'softball:' index.html` returns >= 1 AND `grep -c 'basketball:' index.html` returns >= 1 AND `grep -c 'soccer:' index.html` returns >= 1
    - `grep -c 'data-render="run-standings"' index.html` returns exactly 1 (no dup)
    - All inline `<script>` blocks parse via `node --check` — 0 errors
    - Browser DOM check (DevTools): `document.querySelectorAll('#run-standings tbody[data-render="run-standings"] tr').length >= 4`
    - Browser DOM check: `document.querySelector('#run-standings tr.ours-row').textContent.includes('Diamonds')` is true
    - DevTools console clean on page load
  </acceptance_criteria>
</task>

</tasks>

<verification>
1. Reload `http://localhost:8765/`. Console clean.
2. Expand Step 4. Standings card populated with softball rows. Diamonds highlighted.
3. DevTools console: `window.renderRunStandings('basketball')` swaps to basketball with Breakers ours-row. (Then re-call with 'softball' to leave clean.)
4. `git diff --stat index.html` shows one file changed.
</verification>

<output>
Write `.planning/phases/01-lift-the-widget/01-lift-the-widget-03-SUMMARY.md` per template.
</output>
