Codebase (current state): https://github.com/Abepena/leag

Please read and address the following:

---

# Demo artifact prompt — Family calendar revamp (`#surfaces`)

You are revamping an existing landing-page demo at `#surfaces` titled "What families see all season." It currently shows a static June calendar grid + a side panel with a "Jun 17 family update" card. The grid has dot markers per day, a selected day with a green outline, a venue table below, and a metric strip (start time, family count, alerts) at the bottom-right.

This revamp replaces the static screenshot-style demo with an interactive parent-facing dashboard that demonstrates how a family experiences the season day-to-day.

## Goal

A two-pane layout. Left: month calendar with event-type dots, click a day to select. Right: family update card driven by selection, with live status, venue, arrival note, and any change. Bottom-left: matchup table (current week) so a parent can see "what's coming up". Bottom-right: stat strip showing live game state when applicable. Everything reacts to the day clicked on the calendar.

## Brand voice

- Direct, technical, no corporate jargon. Avoid "leverage", "utilize", "comprehensive", "robust", "seamless".
- Founder voice. "We" for LEAG, "you" for the parent.
- No emojis. No em-dashes (`—`) or en-dashes (`–`). Hyphens fine.
- Sentence case headings.
- No competitor names in copy.
- Numbers as digits.

## Style tokens (already on `:root` in index.html)

```css
--bg: #0c1616; --surface: #101918; --surface-2: #141e1c; --surface-3: #1b2523;
--ink: #f4f5f4; --muted: rgba(244,245,244,0.72); --quiet: rgba(244,245,244,0.48);
--line: rgba(244,245,244,0.12); --line-strong: rgba(244,245,244,0.24);
--green: #62b77c; --green-hi: #7dca96; --green-soft: rgba(98,183,124,0.14);
--gold: #c9a24b;
--demo-primary: #3667B5;  /* tenant theme primary, picker-driven */
--demo-accent: #70ECF0;   /* tenant theme accent, picker-driven */
--radius: 8px;
--font-sans: "Geist", sans-serif;
--font-mono: "JetBrains Mono", monospace;
--font-display: "Anton", sans-serif;
```

Selected day, "Game is live" pill, primary stat numbers, family-update card title accent: `var(--demo-primary)`. Live-state highlight on calendar (the day with a live game): `var(--demo-accent)` ring around the day cell. Practice-day dots: `var(--green)`. Tournament/special dots: `var(--gold)`. Cancelled/conflict dots: kept neutral `var(--muted)` (do NOT use `var(--red)`). Status pill backgrounds use these same tokens.

## Section structure

```html
<section class="section" id="surfaces" aria-labelledby="surfaces-title">
  <div class="container">
    <div class="section-head">
      <div class="section-kicker">Calendar</div>
      <h2 id="surfaces-title">What families see all season.</h2>
      <p>Schedule, scores, venues, and last-minute changes. One place, every device.</p>
    </div>
    <div class="demo-stage">
      <div class="fc-grid">
        <div class="fc-cal-pane">
          <div class="fc-cal-head"><!-- "June schedule" + filter chip --></div>
          <div class="fc-cal-grid"><!-- 5x7 day cells --></div>
          <div class="fc-cal-table"><!-- date / matchup / venue / status --></div>
        </div>
        <aside class="fc-detail-pane">
          <div class="fc-detail-head"><!-- "Jun 17 family update" + Live pill --></div>
          <article class="fc-detail-card"><!-- matchup, venue, time, body copy --></article>
          <ul class="fc-detail-meta"><!-- Game is live primary / Runners on 1st and 3rd note --></ul>
          <div class="fc-detail-stats"><!-- 5:30 start / 18 families / 2 alerts --></div>
        </aside>
      </div>
    </div>
  </div>
</section>
```

## Sample data canon

**Tenant org:** Bayside County Sports / Diamonds 16U.
**Filter chip displayed:** `Filter: 16U / home games` (mono, top-right of calendar pane). Click to cycle through `16U / home games` → `16U / all games` → `All Diamonds`.
**Month displayed:** June (4 full weeks visible: Jun 1-28). Day-of-week header: M T W T F S S.

### Day-cell data (June)

| Date | Type | Detail |
|---|---|---|
| 3 | Practice | Practice / Field 2 / 5:30 PM |
| 5 | Tournament | Bayside Cup Day 1 / Main complex |
| 8 | Practice | Practice / Field 2 / 5:30 PM |
| 10 | Tournament | Bayside Cup Day 2 / Main complex |
| 16 | Practice | Practice / Field 2 / 5:30 PM |
| 17 | Game (live default) | Diamonds vs Marlins / Field 1 / 5:30 PM |
| 19 | Game | Comets vs Hammerheads / Field 4 / 6:45 PM |
| 24 | Bracket | Semifinal bracket opens / Main complex |

Other days are empty.

**Default selected:** Jun 17 (game day, currently live in demo).

### Bottom matchup table (driven by selection week)

Show the 4 matchups visible in the week of the selected day. Default (week of Jun 16-22):

| Date | Matchup | Venue | Status |
|---|---|---|---|
| Jun 16 | Diamonds practice | Field 2 | Scheduled |
| Jun 17 | Diamonds vs Marlins | Field 1 | Live |
| Jun 19 | Comets vs Hammerheads | Field 4 | Scheduled |
| Jun 24 | Semifinal bracket opens | Main complex | Open |

Status pills:
- Scheduled: `var(--green-soft)` background, `var(--green-hi)` text
- Live: `var(--demo-primary)` background, white text, with `var(--demo-accent)` left-edge bar
- Open: faint `var(--surface-3)` background, `var(--quiet)` text

### Detail card (right pane, driven by selection)

For Jun 17 (default):
- Title: `Diamonds vs Marlins`
- Sub-line: `Field 1 / 5:30 PM`
- Body copy: `Families see the game status, venue, arrival note, and any change without scrolling through a roster.`
- Meta list:
  - `Game is live` → tagged "primary" pill on right (`var(--demo-primary)`)
  - `Runners on 1st and 3rd` → tagged "note" pill on right (faint)
- Stat strip:
  - `5:30` start (display font, italic)
  - `18` families (display font, italic)
  - `2` alerts (display font, italic)

For non-game days (e.g., Jun 3 practice), detail card swaps to:
- Title: `Diamonds practice`
- Sub-line: `Field 2 / 5:30 PM`
- Body copy: `Practice notes go here. Coach posts last-minute changes. Families get a push if anything moves.`
- Meta list: `On schedule` (primary pill) / `Bring water bottles` (note pill).
- Stat strip: `5:30 start / 14 families / 0 alerts`.

For empty days (e.g., Jun 4): detail card shows: "No Diamonds events this day. Click another day to see what's coming." (No meta list, no stats.)

## Demo behavior

### Day cells

- Click a day → selects (border `var(--green)`, label `SELECTED` mono caption underneath the dot, body content of right pane re-renders).
- Selected state animates 120ms (transform-scale 1.0 → 1.04 → 1.0).
- Each event-type day has a colored bar/dot under the date number:
  - Practice: small `var(--green)` bar
  - Tournament: `var(--gold)` bar
  - Game: `var(--demo-primary)` bar
  - Bracket: `var(--demo-accent)` bar
- Day with the live game (Jun 17 in default) has a faint `var(--demo-accent)` ring around the cell pulsing 1.6s ease-in-out infinite.
- Hover any day: `var(--surface-2)` background fill.

### Filter chip

- Click cycles through 3 filter modes; calendar dots reduce/restore based on filter:
  - `16U / home games` (default): only show Jun 17 game + Jun 24 bracket
  - `16U / all games`: show Jun 17 + Jun 19 + Jun 24
  - `All Diamonds`: show everything

### Bottom matchup table

- Click any row in the table: same effect as clicking the matching day cell (right pane updates).
- Selected row in table: `var(--green-soft)` background, left-edge `var(--green)` bar.

### Detail pane (right)

- Lives entirely off the selected day.
- "Jun 17 family update" headline updates the date and sub-text reflective of selection.
- "Live" pill in `var(--demo-primary)` shows only when the selected day has an in-progress game (just Jun 17 in demo).
- Stats strip values morph (using a CountUp effect, 400ms) when selection changes.

### Demo footnote (below the stage)

`Demo only. Nothing saves, sends, or charges. Real calendar updates push to every family the moment the league enters the change.`

## Accessibility

- Calendar grid is `role="grid"` with `role="row"` per week, `role="gridcell"` per day, `aria-selected="true"` on the selected day.
- Filter chip is a `<button>` with `aria-label="Filter calendar, current: 16U / home games"` updated on cycle.
- Detail pane uses `aria-live="polite"` so selection-driven updates are announced.
- Day cells have `aria-label="June 17, Diamonds vs Marlins at Field 1, 5:30 PM, live"`.
- Matchup-table rows are real `<tr>` with `<button>`-ified date cells (or `tabindex="0"` + Enter handler).

## Mobile (360px+)

- Calendar pane stays on top, detail pane below (column flow).
- Calendar cells shrink: dot bars stay visible; weekday headers truncate to single letter.
- Matchup table collapses to a vertical card list (one card per matchup).
- Stats strip stays 3-up; numbers shrink to ~28px display.

## Acceptance criteria

- [ ] All 8 event days render with correct color-coded bars in default filter.
- [ ] Filter chip cycles through 3 states and visibly hides/restores correct day markers.
- [ ] Click any day updates detail pane (title, sub-line, body, meta list, stats).
- [ ] Click matchup-table row mirrors day-cell selection behavior.
- [ ] Live pulse ring stays on Jun 17 regardless of selection (until status changes).
- [ ] Stats CountUp animation triggers on selection change.
- [ ] Theme picker re-tints selected-day border, Live pill, primary stats, calendar dots that use demo-primary.
- [ ] No external deps. No console errors. Renders at 360px and 1180px.

## Output format

Return a single self-contained section snippet:
1. The `<section class="section" id="surfaces">` wrapper (preserve the existing `id="surfaces"` and `aria-labelledby="surfaces-title"` so the nav anchor keeps working).
2. Inline `<style>` block scoped to classes prefixed `fc-` to avoid leaking.
3. Inline `<script>` for selection + filter cycle + CountUp (vanilla JS, no deps).
4. Uses `var(--demo-primary)` / `var(--demo-accent)` for tenant accents. Zero use of `var(--red)`.

Do NOT output a full HTML document. This snippet replaces the existing `#surfaces` section in `index.html`.
