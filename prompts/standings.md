Codebase (current state): https://github.com/Abepena/leag

Please read and address the following:

---

# Demo artifact prompt — Standings (`#standings`)

You are designing a single interactive demo section for the LEAG landing page (leag.app), anchored at `#standings`. Demonstrates a sortable division-standings table with click-to-highlight + last-3-results sparkline.

## Goal

Show a clean standings table for one division, 6 teams. Click a column header to sort. Click a team row to expand a last-3-results micro-strip below the row.

## Brand voice

- Direct, technical, no corporate jargon. Avoid "leverage", "utilize", "comprehensive", "robust".
- Founder voice. "We" for LEAG, "you" for the spectator/coach.
- No emojis. No em/en-dashes. Hyphens fine.
- Sentence case headings.
- Numbers as digits.

## Style tokens (already on `:root` in index.html)

```css
--bg: #0c1616; --surface: #101918; --surface-2: #141e1c; --surface-3: #1b2523;
--ink: #f4f5f4; --muted: rgba(244,245,244,0.72); --quiet: rgba(244,245,244,0.48);
--line: rgba(244,245,244,0.12); --line-strong: rgba(244,245,244,0.24);
--green: #62b77c; --green-hi: #7dca96;
--demo-primary: #3667B5;  /* tenant theme primary, picker-driven */
--demo-accent: #70ECF0;   /* tenant theme accent, picker-driven */
--radius: 8px;
--font-sans: "Geist", sans-serif;
--font-mono: "JetBrains Mono", monospace;
--font-display: "Anton", sans-serif;
```

The home-org row (Diamonds, since it's Bayside's softball team) is tinted with `var(--demo-primary)` (a subtle left-edge bar + primary-tinted team name). Win cells and W column header use `var(--demo-primary)`. Loss cells stay neutral. Last-3-results sparkline uses `var(--demo-accent)` for wins and a neutral `var(--quiet)` color for losses.

## Section structure

```html
<section class="demo-section" id="standings">
  <div class="container">
    <div class="section-head">
      <div class="section-kicker">Standings</div>
      <h2 id="standings-title">Standings every parent can read.</h2>
      <p>One division, sortable columns, last 3 results inline. Updates the moment the home-plate score lands.</p>
    </div>
    <div class="demo-stage">
      <div class="standings-meta">
        <span class="mono">Bayside / 12U Spring Softball / National Division</span>
        <span class="mono">Updated live</span>
      </div>
      <table class="standings-table"><!-- 6 teams --></table>
    </div>
  </div>
</section>
```

## Sample data canon

**Tenant org:** Bayside County Sports / Diamonds.
**Six teams in the division:**

| # | Team | W | L | Pct | GB | RS | RA | Last 3 |
|---|---|---|---|---|---|---|---|---|
| 1 | Diamonds | 12 | 3 | .800 | - | 88 | 41 | W W W |
| 2 | Comets | 10 | 5 | .667 | 2.0 | 71 | 52 | W L W |
| 3 | Marlins | 8 | 7 | .533 | 4.0 | 64 | 60 | L W W |
| 4 | Hammerheads | 7 | 8 | .467 | 5.0 | 58 | 65 | W L L |
| 5 | Tide | 5 | 10 | .333 | 7.0 | 49 | 78 | L L W |
| 6 | Stingrays | 3 | 12 | .200 | 9.0 | 38 | 84 | L L L |

**Diamonds is the home org** — its row is highlighted (left bar in `var(--demo-primary)`, team name colored with `var(--demo-primary)`, subtle `var(--demo-primary)` 8% background tint).

## Demo behavior

### Initial state

- Sorted by Pct descending (Diamonds first, Stingrays last).
- "Last 3" column shows three small squares per team: W = filled `var(--demo-accent)`, L = outlined `var(--quiet)`. Most-recent on the right.

### Sort interaction

- Click any column header (W, L, Pct, GB, RS, RA): sort ascending. Click again: sort descending. `aria-sort` toggles `ascending` / `descending` / `none`.
- Active sort header gets a small caret (▲ / ▼) in `var(--demo-primary)`.
- Sort animates rows reordering via FLIP technique (~200ms transform).

### Row expand

- Click any team row: expand a thin sub-row below it showing 3 mini game cards for the last 3 games:
  - "vs Hammerheads / W 6-2 / Field 1 / Apr 18"
  - "@ Marlins / L 4-7 / Field 3 / Apr 15"
  - "vs Comets / W 5-1 / Field 1 / Apr 12"
- Expanded row scrolls into view. Re-clicking collapses. Only one row expanded at a time.

### Highlight interaction

- Hovered row gets `var(--surface-3)` background.
- Diamonds row stays primary-tinted regardless of sort/scroll.

### Demo footnote

Below the stage: "Demo only. Nothing saves, sends, or charges. Real standings update from real scores."

## Accessibility

- `<table>` with `<caption class="sr-only">Bayside 12U Spring Softball National Division standings</caption>`.
- Each sortable header is a `<button>` inside `<th>` with `aria-sort` toggling.
- Expanded sub-row uses `aria-live="polite"` so screen readers announce the last-3 details.
- Last-3 squares each have `aria-label="Win"` / `aria-label="Loss"` (not just visual).

## Mobile (360px+)

- Hide RS / RA columns at narrow widths (CSS `@media`); add a toggle "Show advanced stats" that reveals them.
- Last-3 sparkline stays visible (small).
- Team names truncate with ellipsis if needed; tap-target stays 44px tall.

## Acceptance criteria

- [ ] All 6 teams render with correct data.
- [ ] Sort works on each column (W, L, Pct, GB, RS, RA). Default Pct desc.
- [ ] Diamonds row highlighted regardless of sort (primary left-bar + primary name color).
- [ ] Click row expands last-3-games sub-row with game details. Only one expanded at a time.
- [ ] Last-3 squares: wins fill with `var(--demo-accent)`, losses outlined neutral.
- [ ] Theme picker on the playground re-tints Diamonds-row + win cells live.
- [ ] No external deps. No console errors. Renders at 360px and 1180px.

## Output format

Return a single self-contained section snippet:
1. The `<section class="demo-section" id="standings">` wrapper
2. Inline `<style>` block scoped to classes prefixed `standings-` to avoid leaking
3. Inline `<script>` for sort + expand (vanilla JS, no deps)
4. Uses `var(--demo-primary)` / `var(--demo-accent)` for tenant accents

Do NOT output a full HTML document. The snippet drops into `index.html` after the existing `#publish-schedule` section.
