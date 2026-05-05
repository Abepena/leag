---
phase: 01-lift-the-widget
plan: 01
type: execute
wave: 1
depends_on: []
files_modified:
  - index.html
autonomous: true
requirements:
  - STEP4-01
  - STEP4-02

must_haves:
  truths:
    - "Step 4 accordion body shows the lifted scoreboard widget instead of the old basic scorekeeper."
    - "Below the scoreboard, the standings card renders with eyebrow + title + table, lock-foot stripped."
    - "Both wrapped together in a single .step4-stack container inside the existing .accordion-panel."
    - "Scoreboard widget under #run-scoreboard, scoped CSS does not collide with hero .score-hero."
    - "All scoreboard CSS prefixed with #run-scoreboard."
  artifacts:
    - path: "index.html"
      contains: 'id="run-scoreboard"'
    - path: "index.html"
      contains: 'data-render="run-standings"'
    - path: "index.html"
      contains: 'class="step4-stack"'
---

<objective>
Lift the scoreboard + standings widgets from `_preview/organize.html` into Step 4 accordion body in `index.html`. Replace existing basic-scorekeeper markup. Add scoped CSS prefixed under `#run-scoreboard` / `#run-standings`. Strip standings lock-foot. Do NOT touch JS in this plan.
</objective>

<context>
Step-4 region in index.html:
- Outer `<section class="accordion-item" data-panel="run">` line 6094 — KEEP
- Trigger button + `<span class="status-tag" id="score-status">live</span>` lines 6095-6102 — KEEP
- `<div class="accordion-panel">` line 6103 — KEEP
- Inner `<div class="mini-demo-grid">...</div>` block lines 6104-6147 — REPLACE
- Closing `</div></section>` lines 6148-6149 — KEEP

Source ranges in `_preview/organize.html`:
- Scoreboard HTML: lines 6658-6840
- Scoreboard CSS: lines ~5319-5678
- Standings HTML: lines 6860-6886 (strip lock-foot)

Insert CSS just before first `</style>` in index.html (find via `grep -n '</style>' index.html | head -1`).

CSS scoping: prefix every selector with `#run-scoreboard ` (scoreboard rules) or `#run-standings ` (standings rules). Keyframes — don't prefix the @keyframes name; if name is generic like `pulse`, rename to `runScoreboardPulse` and update animation: refs.
</context>

<tasks>

<task type="auto">
  <name>Task 1: Replace Step 4 body with lifted scoreboard + standings markup</name>
  <read_first>
    - index.html (lines 6094-6155 to confirm current step-4 region)
    - _preview/organize.html (lines 6658-6840 scoreboard, 6860-6886 standings)
  </read_first>
  <files>index.html</files>
  <action>
Replace ONLY lines 6104-6147 (the `<div class="mini-demo-grid">...</div>` block) with this new structure:

```html
                          <div class="step4-stack">
                            <div id="run-scoreboard" class="acc-demo-host">
                              <div class="demo-stage scores-stage">
                                <!-- Lifted from _preview/organize.html lines 6658-6840 -->
                                <!-- Drop outer <section class="section"> and <div class="container"> wrappers -->
                                <!-- Keep <div class="scores-board"> and all children verbatim -->
                                <!-- Keep all data-action, data-count-action, data-score-final, data-score-reset attributes -->
                              </div>
                            </div>
                            <article class="card" id="run-standings">
                              <!-- Lifted from _preview/organize.html lines 6860-6886 -->
                              <!-- KEEP: card-head (eyebrow + title), card-body, table.mini-table, thead, tbody -->
                              <!-- STRIP: any .lock-foot element -->
                              <!-- RENAME: tbody data attribute from data-render="ov-standings" → data-render="run-standings" -->
                            </article>
                          </div>
```

Steps:
1. Read `_preview/organize.html` lines 6658-6840. Identify `<div class="scores-board">...</div>` boundary.
2. Read `_preview/organize.html` lines 6860-6886. Identify `<article class="card">` standings. Strip `.lock-foot` element.
3. Rename standings tbody attribute to `data-render="run-standings"`.
4. Replace lines 6104-6147 of index.html.
  </action>
  <acceptance_criteria>
    - `grep -c 'id="run-scoreboard"' index.html` returns exactly 1
    - `grep -c 'id="run-standings"' index.html` returns exactly 1
    - `grep -c 'data-render="run-standings"' index.html` returns exactly 1
    - `grep -c 'class="step4-stack"' index.html` returns exactly 1
    - `awk 'NR>=6094 && NR<=6160' index.html | grep -c 'data-score-action'` returns 0
    - `awk 'NR>=6094 && NR<=6200' index.html | grep -c 'lock-foot'` returns 0
    - `grep -c 'id="score-status"' index.html` returns exactly 1 (chip preserved)
    - All hero IDs untouched: `grep -c 'id="hero-score-diamonds"'`, `grep -c 'id="hero-score-hammerheads"'`, `grep -c 'id="hero-line-diamonds-total"'`, `grep -c 'id="hero-line-hammerheads-total"'`, `grep -c 'id="hero-inning-label"'` each returns exactly 1
  </acceptance_criteria>
</task>

<task type="auto">
  <name>Task 2: Append scoped CSS for #run-scoreboard and #run-standings</name>
  <read_first>
    - index.html (read around `</style>` to find insertion point; read `.acc-demo-host` rules ~lines 4220-4234; read `.step3-stack` for pattern ~line 1064)
    - _preview/organize.html (lines 5319-5678 scoreboard CSS + nearby standings CSS)
  </read_first>
  <files>index.html</files>
  <action>
1. Find first `</style>` via `grep -n '</style>' index.html | head -1`. Insert blocks immediately before that.

2. Block A — scoreboard CSS prefixed with `#run-scoreboard `:
   - Copy rules from `_preview/organize.html` ~5319-5678 matching: `.scores-*`, `.score-*` (excluding hero `.score-hero`), `.linescore`, `.live-pill`, `.score-button-row`, `.score-control-group`, `.scores-stage`, `.scores-recap`, `.scores-controls`, `.scores-board`.
   - Prefix every top-level selector: `.scores-board { ... }` → `#run-scoreboard .scores-board { ... }`
   - For media queries, prefix inner selectors only.
   - For `@keyframes`, don't prefix; rename generic names like `pulse` to `runScoreboardPulse` and update animation refs.

3. Block B — standings CSS prefixed with `#run-standings `:
   - Copy: `.mini-table`, `.mini-table thead/th/td`, `.team-chip`, `.team-swatch` and variants (`.softball`, `.basketball`, `.soccer`, `.opp`, `.t-1` through `.t-6`), `.ours-row`, `.ours`.
   - Prefix every selector with `#run-standings `.

4. Add `.step4-stack` rule (no prefix) modeled after `.step3-stack`: `display: flex; flex-direction: column; gap: 24px;` (or match step3-stack exactly).

5. Order: `.step4-stack` rule first, then Block A, then Block B. All append-only.
  </action>
  <acceptance_criteria>
    - `grep -c '#run-scoreboard \.scores-board' index.html` returns >= 1
    - `grep -c '#run-scoreboard \.score-hero' index.html` returns >= 1
    - `grep -c '#run-scoreboard \.live-pill' index.html` returns >= 1
    - `grep -c '#run-scoreboard \.linescore' index.html` returns >= 1
    - `grep -c '#run-standings \.mini-table' index.html` returns >= 1
    - `grep -c '#run-standings \.ours-row' index.html` returns >= 1
    - `grep -c '\.step4-stack' index.html` returns >= 2 (HTML + CSS)
    - CSS braces balance: `python3 -c "import re; css=re.search(r'<style>(.*?)</style>',open('index.html').read(),re.DOTALL).group(1); print('balanced' if css.count('{')==css.count('}') else 'IMBALANCED')"` prints `balanced`
  </acceptance_criteria>
</task>

</tasks>

<verification>
Reload `http://localhost:8765/`. Expand Step 4. Lifted scoreboard appears (LIVE pill, big scores, linescore, controls). Standings card below (eyebrow + title + headers, empty rows). Old scorekeeper gone. Hero scoreboard intact (no JS yet — markup-only).
</verification>

<output>
Write `.planning/phases/01-lift-the-widget/01-lift-the-widget-01-SUMMARY.md` per template.
</output>
