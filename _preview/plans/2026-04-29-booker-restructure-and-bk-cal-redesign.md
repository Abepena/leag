# Booker restructure + bk-cal redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure the coach Booking page in `_preview/organize.html` so the coach-facing editor (coach-dash card) owns scheduling, default fee, default availability, and per-event payout; the parent-facing preview (booker-frame) is a clean "What they see" surface. Then visually re-skin the bk-cal grid to match the cb-panel-priv day-stacked aesthetic.

**Architecture:**
- Single HTML file (`_preview/organize.html`) — preview/demo for the LEAG dashboard. No build step.
- Two related surfaces: `#coach-dash` (coach editor) and `#coach-booking.booker-frame` (parent preview). They share state via shared JS data store and the `--demo-primary` / `--demo-accent` theme variables.
- Restructure plan moves the fee toggle OUT of the parent-preview coach-card aside and INTO the coach-dash header. Adds a Default weekly availability collapsible above bk-rates. Adds remove/unblock affordances. Converts rate model from per-30-min to per-hour with auto half-hour calc. Re-skins bk-cal grid to cb-priv-grid layout (7 day columns, stacked time-slot tiles, italic day-num headers, cb-slot tile style).

**Tech Stack:** Plain HTML + CSS + vanilla JS in a single static page. Geist + JetBrains Mono + Anton fonts. Theme via CSS custom properties on `:root`.

---

## File Structure

All work in one file:

- Modify: `_preview/organize.html` — single-file dashboard preview. Sections touched (line numbers approximate, will shift):
  - `<style>` block: `#coach-dash` rules (~lines 1100-1400), `#coach-booking` rules (~lines 1760-2960), shared cb-* / bk-* atoms.
  - HTML body: `<section id="pane-booking">` (~lines 7144-7402) — coach-dash card + booker-frame.
  - JS module at bottom of file: bk-cal renderer (~line 9415), cb-priv renderer, fee-toggle handler (~line 9079), payout calc (~line 9072).

No new files. Plan-doc lives in `_preview/plans/`.

---

## Task 1: Move fee toggle out of coach card aside, drop the `cb-payout-card` from parent preview

**Why first:** Establishes that the parent preview is fee-mode-agnostic (just shows price). Unblocks coach-side payout work.

**Files:**
- Modify: `_preview/organize.html` lines ~7298-7308 (cb-fee-toggle inside cb-coach-card aside) — DELETE.
- Modify: `_preview/organize.html` lines ~8073-8090 (cb-payout-card inside step form / privates panel) — DELETE.
- Modify: `_preview/organize.html` JS — remove fee-mode wiring that reads/writes from cb-coach-card; remove payout-card show/hide logic referencing cb-payout-card.

- [ ] **Step 1: Delete the cb-fee-toggle block from the coach card aside**

Locate this block in `_preview/organize.html` (~line 7298):

```html
<div class="cb-fee-toggle" role="group" aria-label="Service fee handling">
  <div class="cb-fee-toggle-head">
    <span class="cb-fee-toggle-label">Signup checkout fee</span>
    <span class="cb-fee-toggle-rate">5% + $0.50</span>
  </div>
  <div class="cb-fee-toggle-row" role="radiogroup" aria-label="Who pays the service fee">
    <button type="button" class="cb-fee-chip is-active" role="radio" aria-checked="true" data-fee-mode="external">Parent pays</button>
    <button type="button" class="cb-fee-chip" role="radio" aria-checked="false" data-fee-mode="internal">Coach absorbs</button>
  </div>
  <div class="cb-fee-toggle-note" id="cb-fee-note">Parent sees sticker + fee. You receive sticker.</div>
</div>
```

Delete it. Save.

- [ ] **Step 2: Delete the cb-payout-card block from the privates panel**

Locate (~line 8073):

```html
<div class="cb-payout-card" id="cb-payout-card" hidden>
  <div class="cb-payout-eyebrow">Coach payout</div>
  <div class="cb-payout-row">
    <span>Service fee</span>
    <b id="cb-payout-fee">- $2.75</b>
  </div>
  <div class="cb-payout-row cb-payout-net">
    <span>Net payout</span>
    <b id="cb-payout-net">$32.75</b>
  </div>
</div>
```

Delete it. Save.

- [ ] **Step 3: Strip JS that updates cb-payout-card / cb-fee-note from inside the coach card**

Find the JS function that toggles `payoutCard.hidden = !absorb;` (~line 9079). Replace with:

```js
// Fee mode now lives in coach-dash header — see Task 4.
// No payout card in the parent-facing preview.
```

Find references to `#cb-fee-note` and the `cb-fee-toggle-row` event listeners. Move the fee-mode state into a single shared variable, e.g. `state.feeMode = 'external' | 'internal'` (default `'external'`). Keep the click handler logic but rebind in Task 4 to the new chips inside coach-dash.

- [ ] **Step 4: Open `_preview/organize.html` in a browser and verify**

Run: `open _preview/organize.html`
Expected:
- Coach card aside in the booker-frame no longer shows the fee toggle.
- Privates panel no longer shows the "Coach payout" card.
- No JS console errors. Page renders.

- [ ] **Step 5: Commit**

```bash
cd /Users/abe/Desktop/Projects/stryder/products/leag-landing-page
git add _preview/organize.html _preview/plans/2026-04-29-booker-restructure-and-bk-cal-redesign.md
git commit -m "fix(preview): drop fee toggle from coach card aside and payout card from parent step form"
```

---

## Task 2: Add master "What they see" header above the entire booker-frame; drop the eyebrow above offerings

**Files:**
- Modify: `_preview/organize.html` lines ~7250-7397 (booker-frame + cb-offerings-eyebrow).

- [ ] **Step 1: Insert a new master preview header above `<section id="coach-booking" class="booker-frame">`**

Find (~line 7250):

```html
<section id="coach-booking" class="booker-frame" aria-labelledby="coach-booking-title">
```

Replace the line above (`<article class="card" id="coach-dash">...</article>`) so that AFTER coach-dash closes, BEFORE booker-frame opens, you add:

```html
<header class="cb-preview-header">
  <span class="cb-preview-eyebrow-key">Preview</span>
  <h3 class="cb-preview-eyebrow-title">What they see</h3>
  <p class="cb-preview-sub">This is the public booking page parents land on at <code data-coach-url-template="coach.{slug}.leag.app/book">coach.chris.leag.app/book</code>.</p>
</header>
```

- [ ] **Step 2: Remove the inner offerings eyebrow header block**

Find (~line 7322):

```html
<header class="cb-offerings-eyebrow">
  <span class="cb-offerings-eyebrow-key">Preview</span>
  <h3 class="cb-offerings-eyebrow-title">What they see</h3>
</header>
```

Delete it. Save.

- [ ] **Step 3: Add CSS for `.cb-preview-header` (matches eyebrow typography)**

Add to the `#coach-booking` style block (or just below it, with a generic class so it sits above the booker-frame):

```css
.cb-preview-header {
  display: flex; flex-direction: column; gap: 4px;
  padding: 14px 18px;
  margin: 24px 0 0;
  border: 1px solid var(--line);
  border-bottom: 0;
  border-radius: var(--radius) var(--radius) 0 0;
  background: var(--bg-2);
}
.cb-preview-eyebrow-key {
  color: var(--demo-accent);
  font-family: var(--font-mono);
  font-size: 10px;
  letter-spacing: 0.16em;
  text-transform: uppercase;
}
.cb-preview-eyebrow-title {
  margin: 0;
  font-family: var(--font-display);
  font-style: italic;
  font-size: 22px;
  font-weight: 800;
  color: var(--ink);
}
.cb-preview-sub {
  margin: 0;
  color: var(--quiet);
  font-family: var(--font-mono);
  font-size: 11px;
  letter-spacing: 0.04em;
}
.cb-preview-sub code {
  font-family: var(--font-mono);
  background: var(--surface-2);
  border: 1px solid var(--line);
  border-radius: 3px;
  padding: 1px 5px;
  color: var(--ink);
}
/* Round only the bottom corners of booker-frame so it nests under the header */
.cb-preview-header + .booker-frame {
  border-radius: 0 0 var(--radius) var(--radius);
  margin-top: -1px; /* collapse border with header bottom */
}
```

- [ ] **Step 4: Verify in browser**

Run: refresh `_preview/organize.html` in browser, navigate to the Booking tab.
Expected:
- Single "PREVIEW · What they see" header sits above the booker-frame, spanning both columns.
- The previous duplicate eyebrow inside the offerings panel is gone.
- The header and booker-frame visually connect (no double border).

- [ ] **Step 5: Commit**

```bash
git add _preview/organize.html
git commit -m "fix(preview): single 'what they see' header above whole booker-frame"
```

---

## Task 3: Restructure coach-dash card header (two-row layout)

**Why:** The card header now needs to host: title block (top-left), Default fee chip-toggle (top-right), Default weekly availability collapsible trigger (bottom-left), Discard + Save (bottom-right). Two rows separated by a thin border.

**Files:**
- Modify: `_preview/organize.html` lines ~7161-7172 (`#coach-dash` `card-head` block).
- Modify: CSS for `#coach-dash .card-head` (custom override).

- [ ] **Step 1: Replace the existing `<header class="card-head">` block in `#coach-dash`**

Find (~line 7162):

```html
<header class="card-head">
  <div class="titles">
    <div class="eyebrow">Live · Coach dashboard</div>
    <h2 class="card-title">Manage availability and prices</h2>
    <p class="card-sub">Drag across cells to block, add classes, or set custom rates. Preview below updates as you go.</p>
  </div>
  <div class="card-head-actions">
    <button class="btn btn-ghost btn-sm" id="bk-discard" disabled>Discard</button>
    <button class="btn btn-primary btn-sm" id="bk-save" disabled><span id="bk-save-label">Saved</span></button>
  </div>
</header>
```

Replace with:

```html
<header class="card-head bk-card-head">
  <div class="bk-card-head-row bk-card-head-row-top">
    <div class="titles">
      <div class="eyebrow">Live · Coach dashboard</div>
      <h2 class="card-title">Manage availability and prices</h2>
      <p class="card-sub">Drag across cells to block, add classes, or set custom rates. Preview below updates as you go.</p>
    </div>
    <div class="bk-fee-toggle" role="group" aria-label="Default service fee handling">
      <div class="bk-fee-toggle-head">
        <span class="bk-fee-toggle-label">Default fee</span>
        <span class="bk-fee-toggle-rate">5% + $0.50</span>
      </div>
      <div class="bk-fee-toggle-row" role="radiogroup" aria-label="Who pays the service fee by default">
        <button type="button" class="bk-fee-chip is-active" role="radio" aria-checked="true" data-fee-mode="external">Parent pays</button>
        <button type="button" class="bk-fee-chip" role="radio" aria-checked="false" data-fee-mode="internal">Coach absorbs</button>
      </div>
      <div class="bk-fee-toggle-note" id="bk-fee-note">Parent sees sticker + fee. You receive sticker.</div>
    </div>
  </div>
  <div class="bk-card-head-row bk-card-head-row-bottom">
    <button type="button" class="bk-defavail-toggle" id="bk-defavail-toggle" aria-expanded="false" aria-controls="bk-defavail-panel">
      <svg viewBox="0 0 16 16" width="14" height="14" aria-hidden="true">
        <path d="M3 6 L8 11 L13 6" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
      <span class="bk-defavail-toggle-label">Default weekly availability</span>
      <span class="bk-defavail-toggle-summary" id="bk-defavail-summary">Mon-Fri 4-9pm · Sat 9am-2pm · Sun closed</span>
    </button>
    <div class="card-head-actions">
      <button class="btn btn-ghost btn-sm" id="bk-discard" disabled>Discard</button>
      <button class="btn btn-primary btn-sm" id="bk-save" disabled><span id="bk-save-label">Saved</span></button>
    </div>
  </div>
</header>
```

- [ ] **Step 2: Add CSS for the two-row header + fee chips + collapsible trigger**

Add inside the `#coach-dash` style block:

```css
#coach-dash .bk-card-head {
  display: flex; flex-direction: column;
  gap: 0;
  padding: 0;
}
#coach-dash .bk-card-head-row {
  display: flex; align-items: flex-start; justify-content: space-between;
  gap: 16px;
  padding: 16px 18px;
}
#coach-dash .bk-card-head-row-top {
  border-bottom: 1px solid var(--line);
}
#coach-dash .bk-card-head-row-bottom {
  align-items: center;
}

/* Default fee toggle inside coach-dash header (chip pattern from cb-fee-toggle, with overflow fix) */
#coach-dash .bk-fee-toggle {
  display: flex; flex-direction: column; gap: 6px;
  min-width: 240px;
}
#coach-dash .bk-fee-toggle-head {
  display: flex; align-items: baseline; justify-content: space-between; gap: 10px;
}
#coach-dash .bk-fee-toggle-label {
  color: var(--muted);
  font-family: var(--font-mono);
  font-size: 10px;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}
#coach-dash .bk-fee-toggle-rate {
  color: var(--quiet);
  font-family: var(--font-mono);
  font-size: 10px;
  letter-spacing: 0.06em;
}
#coach-dash .bk-fee-toggle-row {
  display: inline-flex;
  border: 1px solid var(--line-strong);
  border-radius: 6px;
  padding: 2px;
  background: var(--bg-2);
}
#coach-dash .bk-fee-chip {
  flex: 1 1 auto;
  min-width: 110px;
  padding: 6px 12px;
  border-radius: 4px;
  background: transparent;
  color: var(--muted);
  font-family: var(--font-sans);
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  white-space: nowrap;
  transition: background 120ms, color 120ms;
}
#coach-dash .bk-fee-chip:hover { color: var(--ink); }
#coach-dash .bk-fee-chip.is-active {
  background: var(--demo-primary);
  color: #fff;
  box-shadow: 0 1px 2px color-mix(in srgb, var(--demo-primary) 40%, transparent);
}
#coach-dash .bk-fee-toggle-note {
  color: var(--quiet);
  font-family: var(--font-mono);
  font-size: 10.5px;
  letter-spacing: 0.04em;
  line-height: 1.4;
}

/* Default weekly availability collapsible trigger */
#coach-dash .bk-defavail-toggle {
  display: inline-flex; align-items: center; gap: 8px;
  padding: 8px 12px;
  border: 1px solid var(--line);
  border-radius: 6px;
  background: var(--bg-2);
  color: var(--ink);
  cursor: pointer;
  font-family: var(--font-sans);
  font-size: 12px;
  transition: border-color 120ms;
}
#coach-dash .bk-defavail-toggle:hover { border-color: var(--demo-accent); }
#coach-dash .bk-defavail-toggle[aria-expanded="true"] svg {
  transform: rotate(180deg);
}
#coach-dash .bk-defavail-toggle svg { transition: transform 160ms; }
#coach-dash .bk-defavail-toggle-label {
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  font-size: 10px;
  color: var(--muted);
}
#coach-dash .bk-defavail-toggle-summary {
  color: var(--quiet);
  font-family: var(--font-mono);
  font-size: 11px;
  letter-spacing: 0.04em;
}
```

- [ ] **Step 3: Wire fee-mode chip JS**

Replace the prior cb-fee-toggle handler with one bound to `.bk-fee-chip`:

```js
(function () {
  const chips = document.querySelectorAll('#coach-dash .bk-fee-chip');
  const note  = document.getElementById('bk-fee-note');
  const NOTES = {
    external: 'Parent sees sticker + fee. You receive sticker.',
    internal: 'You eat the fee. Parent sees a clean sticker. Your payout drops by the fee.'
  };
  function setMode(mode) {
    state.feeMode = mode;
    chips.forEach(c => {
      const active = c.dataset.feeMode === mode;
      c.classList.toggle('is-active', active);
      c.setAttribute('aria-checked', active ? 'true' : 'false');
    });
    if (note) note.textContent = NOTES[mode];
    rerenderBkCal(); // updates payout lines on tiles
  }
  chips.forEach(c => c.addEventListener('click', () => setMode(c.dataset.feeMode)));
  setMode(state.feeMode || 'external');
})();
```

(`rerenderBkCal` is defined in Task 9 — leave a stub for now: `function rerenderBkCal(){}`)

- [ ] **Step 4: Verify in browser**

Refresh. Expected:
- Coach-dash card has two-row header.
- Top row: title block left, Default fee chip-toggle right (no "Coach absorbs" overflow).
- Bottom row: Default weekly availability button left, Discard + Save right.
- Clicking a fee chip toggles state and note text.

- [ ] **Step 5: Commit**

```bash
git add _preview/organize.html
git commit -m "feat(preview): coach-dash two-row header with default fee toggle and weekly-avail trigger"
```

---

## Task 4: Default weekly availability collapsible panel (open-hours per day + recurring unavailable bands)

**Files:**
- Modify: `_preview/organize.html` HTML — add `<section id="bk-defavail-panel">` between coach-dash card head and bk-rates.
- Modify: CSS for the panel.
- Modify: JS for collapsing + storing state.

- [ ] **Step 1: Add the collapsible panel HTML right after the card-head `</header>` and before bk-rates**

Insert:

```html
<section class="bk-defavail" id="bk-defavail-panel" hidden aria-labelledby="bk-defavail-title">
  <header class="bk-defavail-head">
    <div>
      <h3 class="bk-defavail-title" id="bk-defavail-title">Default weekly availability</h3>
      <p class="bk-defavail-sub">Set when you work each week. The grid pre-fills these as closed/open. Override per week with Block.</p>
    </div>
  </header>

  <div class="bk-defavail-grid">
    <div class="bk-defavail-row" data-day="mon"><span class="bk-defavail-dow">Mon</span><label class="bk-defavail-time"><span>From</span><input type="time" value="16:00"></label><label class="bk-defavail-time"><span>To</span><input type="time" value="21:00"></label><label class="bk-defavail-closed"><input type="checkbox"> Closed</label></div>
    <div class="bk-defavail-row" data-day="tue"><span class="bk-defavail-dow">Tue</span><label class="bk-defavail-time"><span>From</span><input type="time" value="16:00"></label><label class="bk-defavail-time"><span>To</span><input type="time" value="21:00"></label><label class="bk-defavail-closed"><input type="checkbox"> Closed</label></div>
    <div class="bk-defavail-row" data-day="wed"><span class="bk-defavail-dow">Wed</span><label class="bk-defavail-time"><span>From</span><input type="time" value="16:00"></label><label class="bk-defavail-time"><span>To</span><input type="time" value="21:00"></label><label class="bk-defavail-closed"><input type="checkbox"> Closed</label></div>
    <div class="bk-defavail-row" data-day="thu"><span class="bk-defavail-dow">Thu</span><label class="bk-defavail-time"><span>From</span><input type="time" value="16:00"></label><label class="bk-defavail-time"><span>To</span><input type="time" value="21:00"></label><label class="bk-defavail-closed"><input type="checkbox"> Closed</label></div>
    <div class="bk-defavail-row" data-day="fri"><span class="bk-defavail-dow">Fri</span><label class="bk-defavail-time"><span>From</span><input type="time" value="16:00"></label><label class="bk-defavail-time"><span>To</span><input type="time" value="21:00"></label><label class="bk-defavail-closed"><input type="checkbox"> Closed</label></div>
    <div class="bk-defavail-row" data-day="sat"><span class="bk-defavail-dow">Sat</span><label class="bk-defavail-time"><span>From</span><input type="time" value="09:00"></label><label class="bk-defavail-time"><span>To</span><input type="time" value="14:00"></label><label class="bk-defavail-closed"><input type="checkbox"> Closed</label></div>
    <div class="bk-defavail-row bk-defavail-row-closed" data-day="sun"><span class="bk-defavail-dow">Sun</span><label class="bk-defavail-time"><span>From</span><input type="time" value="09:00" disabled></label><label class="bk-defavail-time"><span>To</span><input type="time" value="14:00" disabled></label><label class="bk-defavail-closed"><input type="checkbox" checked> Closed</label></div>
  </div>

  <div class="bk-defavail-bands">
    <header class="bk-defavail-bands-head">
      <span>Recurring unavailable inside open hours (optional)</span>
      <button type="button" class="btn btn-secondary btn-sm" id="bk-defavail-band-add"><svg width="12" height="12"><use href="#i-plus"/></svg> Add band</button>
    </header>
    <ul class="bk-defavail-band-list" id="bk-defavail-band-list" role="list">
      <!-- empty by default -->
    </ul>
    <p class="bk-defavail-band-empty" id="bk-defavail-band-empty">No recurring breaks. Add one for things like a weekly dinner block.</p>
  </div>
</section>
```

- [ ] **Step 2: Add CSS for the panel**

```css
#coach-dash .bk-defavail {
  border-bottom: 1px solid var(--line);
  padding: 14px 18px 18px;
  background: var(--bg-2);
}
#coach-dash .bk-defavail[hidden] { display: none; }
#coach-dash .bk-defavail-head { margin-bottom: 12px; }
#coach-dash .bk-defavail-title {
  margin: 0 0 4px;
  font-family: var(--font-sans); font-size: 13px; font-weight: 700;
  color: var(--ink); text-transform: uppercase; letter-spacing: 0.08em;
}
#coach-dash .bk-defavail-sub {
  margin: 0; color: var(--quiet);
  font-family: var(--font-mono); font-size: 11px; letter-spacing: 0.04em;
}
#coach-dash .bk-defavail-grid {
  display: grid; grid-template-columns: repeat(7, 1fr); gap: 6px;
  margin-bottom: 14px;
}
#coach-dash .bk-defavail-row {
  display: flex; flex-direction: column; gap: 4px;
  padding: 8px;
  border: 1px solid var(--line);
  border-radius: 6px;
  background: var(--surface);
}
#coach-dash .bk-defavail-row-closed { opacity: 0.7; }
#coach-dash .bk-defavail-dow {
  font-family: var(--font-mono); font-size: 10px;
  letter-spacing: 0.12em; text-transform: uppercase;
  color: var(--muted);
}
#coach-dash .bk-defavail-time {
  display: flex; flex-direction: column; gap: 2px;
  font-family: var(--font-mono); font-size: 9px;
  letter-spacing: 0.08em; text-transform: uppercase;
  color: var(--quiet);
}
#coach-dash .bk-defavail-time input {
  background: var(--bg-2); border: 1px solid var(--line);
  border-radius: 3px; padding: 4px 6px;
  color: var(--ink); font-family: var(--font-mono); font-size: 11px;
  color-scheme: dark;
}
#coach-dash .bk-defavail-time input:disabled { opacity: 0.5; }
#coach-dash .bk-defavail-closed {
  display: inline-flex; align-items: center; gap: 4px;
  font-family: var(--font-mono); font-size: 10px;
  color: var(--muted); cursor: pointer;
}

/* Bands sub-section */
#coach-dash .bk-defavail-bands {
  border-top: 1px dashed var(--line);
  padding-top: 12px;
}
#coach-dash .bk-defavail-bands-head {
  display: flex; align-items: center; justify-content: space-between;
  font-family: var(--font-mono); font-size: 11px;
  color: var(--muted); margin-bottom: 8px;
  letter-spacing: 0.04em;
}
#coach-dash .bk-defavail-band-list {
  list-style: none; margin: 0; padding: 0;
  display: flex; flex-direction: column; gap: 6px;
}
#coach-dash .bk-defavail-band-list:empty + .bk-defavail-band-empty { display: block; }
#coach-dash .bk-defavail-band-empty {
  margin: 0; color: var(--quiet);
  font-family: var(--font-mono); font-size: 11px;
}
```

- [ ] **Step 3: JS for the collapsible toggle + summary text**

```js
(function () {
  const trig = document.getElementById('bk-defavail-toggle');
  const panel = document.getElementById('bk-defavail-panel');
  const summary = document.getElementById('bk-defavail-summary');
  if (!trig || !panel) return;
  trig.addEventListener('click', () => {
    const open = trig.getAttribute('aria-expanded') === 'true';
    trig.setAttribute('aria-expanded', open ? 'false' : 'true');
    panel.hidden = open;
  });
  // Update summary string based on rows
  function refreshSummary() {
    const rows = panel.querySelectorAll('.bk-defavail-row');
    const groups = []; // simple heuristic: group consecutive same-hours days
    let cur = null;
    rows.forEach(r => {
      const dow = r.querySelector('.bk-defavail-dow').textContent;
      const closed = r.querySelector('.bk-defavail-closed input').checked;
      const from = r.querySelector('.bk-defavail-time:nth-of-type(1) input').value;
      const to   = r.querySelector('.bk-defavail-time:nth-of-type(2) input').value;
      const sig = closed ? 'closed' : `${from}-${to}`;
      if (cur && cur.sig === sig) cur.days.push(dow);
      else { cur = { sig, days: [dow] }; groups.push(cur); }
    });
    summary.textContent = groups.map(g => {
      const range = g.days.length === 1 ? g.days[0] : `${g.days[0]}-${g.days[g.days.length-1]}`;
      return g.sig === 'closed' ? `${range} closed` : `${range} ${fmtTimeRange(g.sig)}`;
    }).join(' · ');
  }
  function fmtTimeRange(sig) {
    const [a, b] = sig.split('-');
    return `${fmt12(a)}-${fmt12(b)}`;
  }
  function fmt12(t) {
    const [h, m] = t.split(':').map(Number);
    const am = h < 12; const h12 = ((h + 11) % 12) + 1;
    return `${h12}${m ? ':' + String(m).padStart(2,'0') : ''}${am ? 'am' : 'pm'}`;
  }
  panel.addEventListener('change', refreshSummary);
  refreshSummary();
})();
```

- [ ] **Step 4: Hook 'Add band' button to a no-op stub for the demo**

```js
document.getElementById('bk-defavail-band-add')?.addEventListener('click', () => {
  const list = document.getElementById('bk-defavail-band-list');
  const empty = document.getElementById('bk-defavail-band-empty');
  const li = document.createElement('li');
  li.className = 'bk-defavail-band';
  li.innerHTML = `
    <select aria-label="Day"><option>Wed</option><option>Mon</option><option>Tue</option><option>Thu</option><option>Fri</option><option>Sat</option><option>Sun</option></select>
    <input type="time" value="18:00" aria-label="From">
    <input type="time" value="19:00" aria-label="To">
    <input type="text" placeholder="Label (e.g. dinner)" maxlength="40">
    <button type="button" class="bk-defavail-band-remove" aria-label="Remove band">×</button>
  `;
  list.appendChild(li);
  if (empty) empty.style.display = 'none';
  li.querySelector('.bk-defavail-band-remove').addEventListener('click', () => {
    li.remove();
    if (!list.children.length && empty) empty.style.display = 'block';
  });
});
```

Add CSS for the band rows:

```css
#coach-dash .bk-defavail-band {
  display: flex; gap: 8px; align-items: center;
  padding: 6px 8px;
  border: 1px solid var(--line);
  border-radius: 5px;
  background: var(--surface);
  font-family: var(--font-mono); font-size: 11px;
}
#coach-dash .bk-defavail-band select,
#coach-dash .bk-defavail-band input {
  background: var(--bg-2); border: 1px solid var(--line);
  border-radius: 3px; padding: 4px 6px;
  color: var(--ink); font-family: var(--font-mono); font-size: 11px;
  color-scheme: dark;
}
#coach-dash .bk-defavail-band-remove {
  margin-left: auto; width: 24px; height: 24px;
  border: 1px solid var(--line);
  border-radius: 4px; background: transparent; color: var(--quiet);
  cursor: pointer;
}
#coach-dash .bk-defavail-band-remove:hover {
  border-color: var(--demo-accent); color: var(--demo-accent);
}
```

- [ ] **Step 5: Verify**

Refresh. Click "Default weekly availability" trigger. Panel slides open. Default rows render. Click "Add band" → row appears. Toggle "Closed" on a row → time inputs disable, summary updates. Collapse trigger → panel hides.

- [ ] **Step 6: Commit**

```bash
git add _preview/organize.html
git commit -m "feat(preview): default weekly availability editor (per-day hours + recurring unavailable bands)"
```

---

## Task 5: Convert rate-rules from per-30-min to per-hour

**Files:**
- Modify: `_preview/organize.html` lines ~7175-7194 (bk-rates section).
- Modify: line ~7186 input id `bk-rate-base` value 40.
- Modify: rule-list rendering JS to display "/hour" instead of "/30 min".
- Modify: any data structure / state field referring to "rate per 30 min" — rename to `ratePerHour`.

- [ ] **Step 1: Update HTML labels**

Find:

```html
<label class="bk-rates-base-label">
  <span>Base rate / 30 min</span>
  <span class="bk-rates-base-input">
    <span class="bk-edit-prefix">$</span>
    <input type="number" id="bk-rate-base" value="40" min="0" step="5">
  </span>
</label>
```

Replace with:

```html
<label class="bk-rates-base-label">
  <span>Base rate / hour</span>
  <span class="bk-rates-base-input">
    <span class="bk-edit-prefix">$</span>
    <input type="number" id="bk-rate-base" value="80" min="0" step="5">
  </span>
</label>
```

- [ ] **Step 2: Update the floating-toolbar rate form in the template (~line 7232)**

Find:

```html
<form class="bk-toolbar-form bk-rate-form" data-form="rate" hidden>
  <label class="bk-edit-field bk-edit-num"><span>Rate / 30 min</span><span class="bk-edit-prefix">$</span><input name="rate" type="number" value="40" min="0" step="5"></label>
```

Replace with:

```html
<form class="bk-toolbar-form bk-rate-form" data-form="rate" hidden>
  <label class="bk-edit-field bk-edit-num"><span>Rate / hour</span><span class="bk-edit-prefix">$</span><input name="rate" type="number" value="80" min="0" step="5"></label>
  <p class="bk-rate-form-half" id="bk-rate-form-half">Half hour: <span data-rate-half>$40</span></p>
```

(Add JS to update `[data-rate-half]` whenever the hourly input changes — round to nearest $5.)

- [ ] **Step 3: Update rate-rule list rendering JS**

Find the rendering function for `bk-rate-list`. Wherever it outputs "$N / 30 min", change to:

```js
const rateLine = `$${r.rate} / hour <span class="bk-rate-half-meta">($${roundTo5(r.rate / 2)} / 30 min)</span>`;
```

Add helper:

```js
function roundTo5(n) { return Math.round(n / 5) * 5; }
```

- [ ] **Step 4: Update parent-facing privates display logic**

Find code that shows the privates panel's tier pill (~line 7351):

```html
<span class="cb-priv-tier-key">30-min private</span>
<span class="cb-priv-tier-val">$40</span>
```

Keep AS-IS — parents still book 30-min privates. The tier-val is computed from `state.ratePerHour / 2` rounded to nearest $5. Update the JS that sets these values:

```js
function refreshPrivTier() {
  const hourly = state.ratePerHour;
  const half   = roundTo5(hourly / 2);
  document.querySelector('.cb-priv-tier-val').textContent = `$${half}`;
}
refreshPrivTier();
```

- [ ] **Step 5: Add small CSS for the half-rate meta**

```css
#coach-dash .bk-rate-half-meta {
  color: var(--quiet);
  font-family: var(--font-mono);
  font-size: 10px;
  margin-left: 6px;
}
#coach-dash .bk-rate-form-half {
  margin: 0;
  color: var(--quiet);
  font-family: var(--font-mono);
  font-size: 10.5px;
  letter-spacing: 0.04em;
}
#coach-dash .bk-rate-form-half [data-rate-half] {
  color: var(--ink);
  font-weight: 600;
}
```

- [ ] **Step 6: Verify**

Refresh. Base-rate input shows `/ hour` and `$80`. Add a rule via toolbar — entered hourly rate, half-hour shows below. Privates panel tier pill shows `$40`.

- [ ] **Step 7: Commit**

```bash
git add _preview/organize.html
git commit -m "feat(preview): hourly rate model with auto-derived 30-min price"
```

---

## Task 6: Per-event payout — show "You get $X" line on tiles + breakdown in toolbar; only when Coach Absorbs; rename to "Service fee"

**Files:**
- Modify: `_preview/organize.html` JS — bk-cal class-tile and slot-tile renderers; toolbar rendering.
- Modify: `_preview/organize.html` HTML — toolbar template adds breakdown rows + per-event override toggle.

- [ ] **Step 1: Extend the toolbar template with a breakdown block + override toggle**

Find `<template id="bk-toolbar-tpl">` (~line 7213). Inside, replace the default-row block with:

```html
<template id="bk-toolbar-tpl">
  <div class="bk-toolbar" role="toolbar" aria-label="Selection actions">
    <div class="bk-toolbar-row" data-row="default">
      <span class="bk-toolbar-meta" data-meta>0 cells</span>
      <button type="button" class="bk-toolbar-btn" data-action="block">Block</button>
      <button type="button" class="bk-toolbar-btn" data-action="unblock" hidden>Unblock</button>
      <button type="button" class="bk-toolbar-btn" data-action="class">Add class</button>
      <button type="button" class="bk-toolbar-btn" data-action="remove-class" hidden>Remove class</button>
      <button type="button" class="bk-toolbar-btn" data-action="rate" data-rate-disabled-when-blocked>Set rate</button>
      <button type="button" class="bk-toolbar-btn bk-toolbar-btn-ghost" data-action="cancel" aria-label="Cancel">×</button>
    </div>
    <!-- existing class-form and rate-form unchanged -->
    <div class="bk-toolbar-payout" data-payout hidden>
      <div class="bk-toolbar-payout-row">
        <span>Sticker</span>
        <b data-payout-sticker>$0.00</b>
      </div>
      <div class="bk-toolbar-payout-row">
        <span>Service fee</span>
        <b data-payout-fee>- $0.00</b>
      </div>
      <div class="bk-toolbar-payout-row bk-toolbar-payout-net">
        <span>You get</span>
        <b data-payout-net>$0.00</b>
      </div>
      <div class="bk-toolbar-fee-override" role="radiogroup" aria-label="Override fee for this event">
        <span class="bk-toolbar-fee-key">Fee mode</span>
        <button type="button" class="bk-fee-chip is-active" role="radio" aria-checked="true" data-fee-mode="external">Parent pays</button>
        <button type="button" class="bk-fee-chip" role="radio" aria-checked="false" data-fee-mode="internal">Coach absorbs</button>
      </div>
    </div>
  </div>
</template>
```

- [ ] **Step 2: Add CSS for the payout block inside the toolbar**

```css
#coach-dash .bk-toolbar-payout {
  display: flex; flex-direction: column; gap: 4px;
  padding: 10px 12px;
  border-top: 1px solid var(--line);
  background: var(--bg-2);
  border-radius: 0 0 6px 6px;
}
#coach-dash .bk-toolbar-payout-row {
  display: flex; align-items: center; justify-content: space-between;
  font-family: var(--font-mono); font-size: 11px;
  color: var(--muted);
}
#coach-dash .bk-toolbar-payout-row b {
  color: var(--ink); font-weight: 600;
}
#coach-dash .bk-toolbar-payout-net {
  border-top: 1px dashed var(--line);
  padding-top: 4px; margin-top: 2px;
  font-size: 12px;
}
#coach-dash .bk-toolbar-payout-net b { color: var(--green); font-size: 14px; }
#coach-dash .bk-toolbar-fee-override {
  display: flex; align-items: center; gap: 6px;
  margin-top: 6px;
  flex-wrap: wrap;
}
#coach-dash .bk-toolbar-fee-key {
  color: var(--quiet);
  font-family: var(--font-mono); font-size: 9.5px;
  letter-spacing: 0.12em; text-transform: uppercase;
  margin-right: 4px;
}
#coach-dash .bk-toolbar-fee-override .bk-fee-chip {
  min-width: 96px;
  padding: 4px 10px;
  font-size: 11px;
}
```

- [ ] **Step 3: JS — compute payout and render based on event mode**

Add helper:

```js
const FEE_PCT = 0.05;
const FEE_FIXED = 0.50;
function computePayout(sticker, mode) {
  // mode: 'external' (parent pays) — coach gets sticker
  //       'internal' (coach absorbs) — coach gets sticker - fee
  const fee = +(sticker * FEE_PCT + FEE_FIXED).toFixed(2);
  const net = mode === 'internal' ? +(sticker - fee).toFixed(2) : sticker;
  return { sticker, fee, net };
}
```

Wherever the toolbar opens with a class or private-slot context, populate the payout block when `event.feeMode === 'internal'`:

```js
function renderToolbarPayout(toolbar, evt) {
  const block = toolbar.querySelector('[data-payout]');
  if (!block) return;
  if (evt.feeMode !== 'internal') {
    block.hidden = true;
    return;
  }
  const { sticker, fee, net } = computePayout(evt.price, evt.feeMode);
  block.hidden = false;
  block.querySelector('[data-payout-sticker]').textContent = `$${sticker.toFixed(2)}`;
  block.querySelector('[data-payout-fee]').textContent = `- $${fee.toFixed(2)}`;
  block.querySelector('[data-payout-net]').textContent = `$${net.toFixed(2)}`;
  // Sync override chips with current event mode
  block.querySelectorAll('.bk-fee-chip').forEach(c => {
    const active = c.dataset.feeMode === evt.feeMode;
    c.classList.toggle('is-active', active);
    c.setAttribute('aria-checked', active ? 'true' : 'false');
  });
}
```

When user clicks an override chip in the toolbar, mutate the event's feeMode (not the global default) and re-render the tile.

- [ ] **Step 4: Render the "You get $X" line on tiles when class-event mode is internal**

Update the bk-cal class-block render (search for `bk-cal-class`):

```js
function renderClassBlock(c) {
  const { sticker, net } = computePayout(c.price, c.feeMode);
  const payoutLine = c.feeMode === 'internal'
    ? `<span class="bk-class-payout">You get $${net.toFixed(2)}</span>`
    : '';
  return `
    <div class="bk-cal-class" data-class-id="${c.id}" data-fee-mode="${c.feeMode}">
      <span class="bk-class-title">${escapeHtml(c.title)}</span>
      <span class="bk-class-meta">${c.timeRange} · ${c.cap} cap · $${c.price}</span>
      ${payoutLine}
    </div>
  `;
}
```

CSS:

```css
#coach-dash .bk-class-payout {
  font-family: var(--font-mono); font-size: 9.5px;
  letter-spacing: 0.04em;
  color: rgba(255,255,255,0.85);
  margin-top: 1px;
}
```

For per-slot custom-rate slots in absorb mode, mirror the same approach inside the slot tile (see Task 9).

- [ ] **Step 5: Verify**

Refresh. With Default fee = "Coach absorbs", placed classes show "You get $X". Click a class → toolbar shows breakdown + override chips. Switch override → tile updates inline. Switch default to "Parent pays" → payout line disappears.

- [ ] **Step 6: Commit**

```bash
git add _preview/organize.html
git commit -m "feat(preview): per-event payout line on tiles + breakdown in toolbar (coach-absorb only)"
```

---

## Task 7: Add UI to UNBLOCK a slot and REMOVE a class

**Files:**
- Modify: `_preview/organize.html` JS toolbar action dispatch.

- [ ] **Step 1: Show 'Unblock' when selection is exclusively blocked cells, hide 'Block' and 'Set rate' (rate stays disabled on blocks)**

In the toolbar-show logic (where you currently set the meta count + decide which buttons appear), add:

```js
function configureToolbar(toolbar, selection) {
  const blockBtn   = toolbar.querySelector('[data-action="block"]');
  const unblockBtn = toolbar.querySelector('[data-action="unblock"]');
  const classBtn   = toolbar.querySelector('[data-action="class"]');
  const removeBtn  = toolbar.querySelector('[data-action="remove-class"]');
  const rateBtn    = toolbar.querySelector('[data-action="rate"]');

  const allBlocked = selection.cells.every(c => c.state === 'blocked');
  const onClass    = selection.kind === 'class';

  blockBtn.hidden   = allBlocked || onClass;
  unblockBtn.hidden = !allBlocked || onClass;
  classBtn.hidden   = onClass;
  removeBtn.hidden  = !onClass;
  rateBtn.disabled  = allBlocked; // KEEP RULE: blocked slot can't take a rate
  rateBtn.hidden    = onClass;    // class tiles don't take per-cell rates
}
```

- [ ] **Step 2: Wire the unblock + remove-class action handlers**

```js
toolbar.addEventListener('click', (e) => {
  const btn = e.target.closest('[data-action]');
  if (!btn) return;
  const action = btn.dataset.action;
  if (action === 'unblock') {
    selection.cells.forEach(c => { c.state = 'open'; });
    rerenderBkCal(); closeToolbar();
  } else if (action === 'remove-class') {
    state.classes = state.classes.filter(c => c.id !== selection.classId);
    rerenderBkCal(); closeToolbar();
  }
});
```

- [ ] **Step 3: Add CSS state for disabled buttons**

```css
#coach-dash .bk-toolbar-btn:disabled {
  opacity: 0.4; cursor: not-allowed;
}
```

- [ ] **Step 4: Verify**

Refresh. Drag-select an open range → block. Toolbar closes. Drag-select that same range → toolbar shows "Unblock" (no Block / no Set rate available). Click Unblock → cells return to open. Click an existing class → toolbar shows "Remove class". Click → class disappears.

- [ ] **Step 5: Commit**

```bash
git add _preview/organize.html
git commit -m "feat(preview): unblock + remove-class actions in floating toolbar"
```

---

## Task 8: Hand off bk-cal grid visual redesign to /impeccable

**Why:** Tasks 1-7 are structural / behavioral. The visual transformation of the bk-cal grid (matrix → day-stacked columns, cb-priv-style tiles) is best done with the impeccable design skill, with companion preview, per the user's request.

- [ ] **Step 1: Verify all preceding tasks committed and the page renders cleanly**

```bash
git status
open _preview/organize.html
```

- [ ] **Step 2: Invoke `/impeccable` with target = `section.bk-cal` and reference = `div#cb-panel-priv.cb-panel-priv`**

The impeccable skill should:
- Read the current `#coach-dash .bk-cal*` CSS and HTML.
- Read the `#coach-booking .cb-priv-*` CSS as the visual reference.
- Restructure `bk-cal-grid` from a matrix grid (1 time-axis col + 7 day cols) into 7 day-columns each containing a vertical stack of time-slot tiles, matching `cb-priv-grid` / `cb-priv-day` / `cb-priv-day-head` / `cb-priv-slots` / `cb-slot` patterns.
- Each slot tile shows: time (e.g. "4:00 PM"), state label (open / pending / booked / blocked / closed), and price ONLY when the cell's rate differs from the base rate.
- Day headers use big italic Anton day-num + small mono DOW.
- Legend dots match cb-priv-legend pattern.
- Drag-select still works on the new layout (multi-select within a day or across days).
- Closed cells (default-unavailable) render as solid dim, no hover.
- Blocked cells keep the dashed pattern.
- Class blocks (multi-slot) render as a single tile spanning their range.
- Theming uses `--demo-primary` and `--demo-accent` exclusively.

- [ ] **Step 3: After /impeccable completes, refresh and verify all behaviors still work**

Smoke test:
- Default fee toggle — chips don't overflow.
- Default weekly availability — collapsible opens/closes.
- bk-cal — drag-select cells → toolbar appears with Block / Add class / Set rate.
- Block a range → cells render dashed; toolbar on re-select shows Unblock.
- Add a class on a range → tile renders; clicking shows "Remove class".
- Coach Absorbs mode — payout line on tiles; toolbar shows breakdown.
- Closed cells (Sun all day, weekday mornings) render solid dim.
- Visual feels like a sibling of cb-panel-priv (typography, spacing, color, legend).

- [ ] **Step 4: Commit**

```bash
git add _preview/organize.html
git commit -m "feat(preview): redesign bk-cal grid to day-stacked cb-priv-style layout"
```

---

## Self-Review

**1. Spec coverage:**

| Spec ask | Covered by task |
|---|---|
| Master "What they see" header above whole booker-frame | Task 2 |
| Bio/stats card excludes signup-fee info | Task 1 (delete) |
| Fee toggle moved to coach-dash header (above schedule) | Task 3 |
| Step form excludes coach payout (always) | Task 1 (delete cb-payout-card) |
| Per-event payout listed when Coach Absorbs only | Task 6 |
| MVP stats: 3 placeholder stats + TBD note | (no change — keep current) |
| Hourly rates with half-hour auto-calc | Task 5 |
| Floating toolbar shows half-hour from hourly | Task 5 |
| Block can't take a rate (KEEP) | Task 7 (rateBtn.disabled when blocked) |
| Unblock UI (FIX) | Task 7 |
| Remove class UI (FIX) | Task 7 |
| Class can be placed over a block (KEEP) | (existing behavior preserved) |
| Default weekly availability (recurring never-available) | Task 4 |
| bk-cal redesign matching cb-panel-priv | Task 8 (handoff to /impeccable) |
| "Service fee" label, not "Stripe fee" | Task 6 |
| Fee chip overflow fix | Task 3 (min-width 110, nowrap, padding 0 12) |

**2. Placeholder scan:** No "TBD" / "implement later" / "fill in details" in concrete-code steps. Stats card gets a one-line code comment marking placeholders, which is the explicit user direction.

**3. Type consistency:** `state.feeMode` ('external' | 'internal') used uniformly across Tasks 3, 6, 7. `event.feeMode` mirrors the same union. `state.ratePerHour` introduced in Task 5 and read in Task 5's `refreshPrivTier`. `roundTo5` helper used in Task 5 and Task 6. `computePayout` defined once in Task 6 and reused in tile + toolbar render paths. `rerenderBkCal` declared as a stub in Task 3 and implemented as part of the existing renderer (Task 8 redesign refines it but keeps the name).

---

## Execution Handoff

Plan complete and saved to `_preview/plans/2026-04-29-booker-restructure-and-bk-cal-redesign.md`.

Two execution options:

1. **Subagent-Driven (recommended)** — I dispatch a fresh subagent per task, review between tasks, fast iteration.
2. **Inline Execution** — Execute tasks in this session using `superpowers:executing-plans`, batch with checkpoints.

Which approach?
