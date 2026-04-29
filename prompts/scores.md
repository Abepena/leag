Codebase (current state): https://github.com/Abepena/leag

Please read and address the following:

---

# Demo artifact prompt — Scores (`#scores`)

You are designing a single interactive demo section for the LEAG landing page (leag.app), anchored at `#scores`. Demonstrates live in-game scoring + a public-facing ticker that auto-scrolls with nearby games.

## Goal

Scoreboard a coach can update one-handed at the field. Click "+1" to bump a team's runs. Click "Next" to advance from Top 5 → Bottom 5 → Final. Final state shows W/L recap. Adjacent ticker auto-scrolls with 4 other Bayside games.

## Brand voice

- Direct, technical, no corporate jargon. Avoid "leverage", "utilize", "comprehensive", "robust".
- Founder voice. "We" for LEAG, "you" for the scorekeeper.
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
--font-display: "Anton", sans-serif;  /* italic uppercase for team names + score numbers */
```

Demo team accents (Diamonds = home in this section) use `var(--demo-primary)`. Live indicator + active inning use `var(--demo-primary)`. Active run cell + base highlights use `var(--demo-accent)`.

**Important:** the existing hero scoreboard component (`.score-hero`, `.team-name`, `.linescore`, `.live-pill`) already uses these vars. Reuse those classes; do not invent new ones.

## Section structure

```html
<section class="demo-section" id="scores">
  <div class="container">
    <div class="section-head">
      <div class="section-kicker">Scores</div>
      <h2 id="scores-title">Score it from the dugout, broadcast it to the bleachers.</h2>
      <p>One scorekeeper updates from a phone. Parents in the stands see the same number their kid sees.</p>
    </div>
    <div class="demo-stage">
      <div class="scores-board">
        <div class="ticker"><!-- mirrors hero ticker --></div>
        <div class="score-hero"><!-- the scoreboard --></div>
        <div class="scores-controls"><!-- +1 / -1 buttons + Next half-inning + Final --></div>
      </div>
    </div>
  </div>
</section>
```

## Sample data canon

**Tenant org:** Bayside County Sports.

**Headline matchup:** Diamonds (home, demo-primary) vs Hammerheads (away, neutral).
**Field:** "Field 1".

**Initial state:**
- Top 5
- Diamonds 5, Hammerheads 3
- Linescore (away top, home bottom):
  - Hammerheads: 1 0 1 1 0 _ _ → 3
  - Diamonds:    0 2 0 3 _ _ _ → 5
- Bases: runner on 1st, runner on 3rd, no outs in 2nd (use the existing diamond SVG markup pattern)

**Ticker rotation (4 nearby games, auto-scroll):**
- "FINAL COMETS 8 - 6 MARLINS"
- "Diamonds 5 - 3 Hammerheads T5"
- "NEXT MON 5:30 FIELD 1"
- "TRYOUTS 12U OPEN"

## Demo behavior

### Scoreboard interactions

- "+1 Diamonds" button: bumps Diamonds total. If currently Bottom of inning, also adds to Diamonds linescore for that inning. If Top of inning, no-op (Diamonds bats in bottom).
- "+1 Hammerheads" button: same logic, mirrored. Hammerheads bats in top.
- "-1" buttons next to each team for corrections. Floor at 0.
- "Next half-inning" button: advances state machine
  - Top 5 → Bottom 5 → Top 6 → Bottom 6 → Top 7 → Bottom 7 → Final
- "Final" button (only enabled in 7th): locks scores, swaps live-pill for "FINAL", reveals W/L recap card below scoreboard with winner highlighted in `var(--demo-primary)`.
- "Reset" button: returns to initial Top 5 state above.

### State indicator

- "LIVE NOW" pill in `var(--demo-primary)` while game in progress.
- Field + half-inning text: "Field 1 / Top 5" updates as state advances.
- Currently-batting team's name uses `.team-name.batting` (demo-primary tint).
- Active inning column in linescore highlights with `var(--demo-primary)` background, white text.

### Ticker

- Horizontal auto-scrolling marquee. CSS `@keyframes scroll`, ~30s loop.
- Pauses on hover (`animation-play-state: paused`).
- Each game span separated by 24px gap, vertical pipe `|` between, mono font.

### Demo footnote

Below the stage: "Demo only. Nothing saves, sends, or charges. Real scoring is one tap on a real phone."

## Accessibility

- Score updates announced via `role="status"` live region: "Diamonds 6, Hammerheads 3, top of 6th".
- Buttons have visible focus rings (2px `var(--demo-primary)` outline).
- Linescore is `role="table"` with `aria-label="Linescore through the {N}th inning"`.
- "+1 Diamonds" reads as "Add 1 to Diamonds" via `aria-label`.

## Mobile (360px+)

- Ticker stays full-width above scoreboard.
- Scoreboard team rows stack: away team row on top, score row in middle, home team row below.
- Score number scales down to ~64px from 96px.
- Controls collapse to: "+1 / -1" pill cluster per team + "Next" + "Final" + "Reset" — sticky bottom bar.

## Acceptance criteria

- [ ] +1 button correctly attributes runs to the batting team for the current half-inning.
- [ ] -1 floors at 0.
- [ ] "Next half-inning" advances correctly through the 7-inning sequence.
- [ ] Active inning highlighting moves with state.
- [ ] Final state shows winner in demo-primary, hides "+1/-1" buttons, exposes Reset.
- [ ] Ticker auto-scrolls and pauses on hover.
- [ ] Bases SVG renders with `fill="var(--demo-accent)"` for occupied bases (do NOT use `var(--red)`).
- [ ] Theme picker on the playground re-tints scoreboard accents live.
- [ ] No external deps. No console errors. Renders at 360px and 1180px.

## Output format

Return a single self-contained section snippet:
1. The `<section class="demo-section" id="scores">` wrapper
2. Inline `<style>` block — reuse class names from the hero scoreboard (`.score-hero`, `.team-name`, `.linescore`, `.live-pill`, `.ticker`) where possible. Scoped overrides prefixed `scores-` for new chrome.
3. Inline `<script>` for the state machine (vanilla JS, no deps)
4. Uses `var(--demo-primary)` / `var(--demo-accent)` for tenant accents. Zero use of `var(--red)`.

Do NOT output a full HTML document. The snippet drops into `index.html` after the existing `#publish-schedule` section.
