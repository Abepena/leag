Codebase (current state): https://github.com/Abepena/leag

Please read and address the following:

---

# Demo artifact prompt — Teams (`#teams`)

You are designing a single interactive demo section for the LEAG landing page (leag.app), anchored at `#teams`. Demonstrates roster + team-builder workflow: drag (or tap) players from an unassigned pool into team rosters, with auto-balance.

## Goal

Show a coach assembling rosters for the season. Three teams ready to receive players, one unassigned pool. Drag-or-tap to assign. "Auto-balance" button distributes evenly. Inline edit of team name. Click a player to expand mini-card with age, jersey #, and skill rating.

## Brand voice

- Direct, technical, no corporate jargon. Avoid "leverage", "utilize", "comprehensive", "robust".
- Founder voice. "We" for LEAG, "you" for the coach.
- No emojis. No em/en-dashes. Hyphens fine.
- Sentence case headings.
- Numbers as digits.

## Style tokens (already on `:root` in index.html)

```css
--bg: #0c1616; --surface: #101918; --surface-2: #141e1c; --surface-3: #1b2523;
--ink: #f4f5f4; --muted: rgba(244,245,244,0.72); --quiet: rgba(244,245,244,0.48);
--line: rgba(244,245,244,0.12); --line-strong: rgba(244,245,244,0.24);
--green: #62b77c; --green-hi: #7dca96; --green-soft: rgba(98,183,124,0.14);
--demo-primary: #3667B5;  /* tenant theme primary, picker-driven */
--demo-accent: #70ECF0;   /* tenant theme accent, picker-driven */
--radius: 8px;
--font-sans: "Geist", sans-serif;
--font-mono: "JetBrains Mono", monospace;
--font-display: "Anton", sans-serif;
```

Tenant team chips, primary CTA, drop-target highlight all use `var(--demo-primary)`. Hover states / drag preview use `var(--demo-accent)`.

## Section structure

```html
<section class="demo-section" id="teams">
  <div class="container">
    <div class="section-head">
      <div class="section-kicker">Teams</div>
      <h2 id="teams-title">Build your rosters in one screen.</h2>
      <p>Drag players in. Auto-balance distributes by skill. Rename a team inline. No spreadsheets.</p>
    </div>
    <div class="demo-stage">
      <div class="teams-controls"><!-- Auto-balance, Reset, # players unassigned --></div>
      <div class="teams-board">
        <div class="teams-pool"><!-- unassigned --></div>
        <div class="teams-rosters"><!-- 3 team columns --></div>
      </div>
    </div>
  </div>
</section>
```

## Sample data canon

**Tenant org:** Bayside County Sports.
**Three teams (rename-able inline):** Diamonds, Breakers, Nets — but the demo is sport-agnostic so call them generic "Team 1 / 2 / 3" by default and let the user click each name to edit. (Pre-fill: "Diamonds Black", "Diamonds White", "Diamonds Gold" — three squads of one club, season-realistic.)

**12 sample players (unassigned pool at start):**

| # | Name | Age | Skill (1-5) |
|---|---|---|---|
| 3 | Bree Walden | 11 | 5 |
| 7 | Sage Foster | 12 | 5 |
| 9 | Kinley Brooks | 11 | 4 |
| 11 | Marlee Quinn | 10 | 3 |
| 14 | Reese Tatum | 12 | 4 |
| 17 | Wren Halliday | 11 | 5 |
| 21 | Avery Pike | 10 | 2 |
| 24 | Harlow Vance | 12 | 3 |
| 32 | Juno Reid | 11 | 3 |
| 44 | Sloane Beck | 12 | 4 |
| 5 | Tatum Cole | 10 | 2 |
| 19 | Rory Fennel | 11 | 3 |

Skill is a 1-5 ranking used for auto-balance. Display as 5 small dots filled per skill level (no stars, no emoji).

## Demo behavior

### Layout

- Left column: "Unassigned" pool of player chips, count badge in header.
- Right column: 3 team cards stacked vertically (or horizontally on wide screens). Each team card has:
  - Editable team name (click to edit, Enter to commit, Esc to cancel)
  - Roster count badge
  - Drop zone for player chips
  - "Average skill" small mono indicator (calculated live)

### Player chip

- Pill: jersey number (mono) / name / skill dots
- Cursor: `grab` on hover, `grabbing` while dragging
- Click (no drag) expands inline mini-card showing age + skill row + "Move to..." dropdown of teams as a tap-fallback for mobile

### Drag-and-drop (desktop)

- Use HTML5 drag-and-drop API. `dragstart` adds `is-dragging` class to chip; `dragover` on a team highlights its border with `var(--demo-accent)`; `drop` reassigns and updates counts + average-skill.
- Drop on Unassigned pool sends a chip back.

### Tap fallback (mobile)

- Tap chip → chip enters "selected" state (outline `--demo-primary`)
- Tap any team card → assigns selected chip to that team
- Tap Unassigned area → returns selected chip to pool

### Controls bar

- "Auto-balance" button (primary, `var(--demo-primary)`): redistributes all players across 3 teams to minimize skill variance. Animate chips moving (200ms transform).
- "Reset" button (secondary): all chips back to Unassigned, team names back to defaults.
- Live readout: "X unassigned, average skill per team: A.A / B.B / C.C".

### Demo footnote

Below the stage: "Demo only. Nothing saves, sends, or charges."

## Accessibility

- Each team card and the Unassigned pool announce as `role="region"` with `aria-label`.
- Player chips are `role="button"` with `aria-grabbed` / `aria-describedby` linking to "press space to pick up, arrow keys to choose target, enter to drop".
- Keyboard: Space picks up a chip. Arrow keys cycle drop targets (highlighted). Enter drops. Esc cancels.
- Live regions announce count changes after each move.

## Mobile (360px+)

- Pool collapses to a horizontal scroller above the team cards.
- Team cards stack full-width.
- "Auto-balance" + "Reset" pinned to a sticky bottom bar.

## Acceptance criteria

- [ ] All 12 sample players present in the pool on initial load.
- [ ] Drag from pool to team works on desktop. Tap-select-tap-target works on mobile.
- [ ] Drag back to pool returns chip to Unassigned.
- [ ] Inline rename of team name works (click, type, Enter commits, Esc cancels).
- [ ] Auto-balance distributes 4 players to each team and minimizes skill-rating variance.
- [ ] Reset returns initial state.
- [ ] Average-skill readout updates live per team after each move.
- [ ] Drop-target highlight uses `var(--demo-accent)`. Selected team accent uses `var(--demo-primary)`.
- [ ] Theme picker on the playground re-tints accents live (no JS plumbing required if you use the CSS vars throughout).
- [ ] No external deps. No console errors. Renders at 360px and 1180px.

## Output format

Return a single self-contained section snippet:
1. The `<section class="demo-section" id="teams">` wrapper
2. Inline `<style>` block scoped to classes prefixed `teams-` to avoid leaking
3. Inline `<script>` for state + drag-drop + keyboard support (vanilla JS, no deps)
4. Uses `var(--demo-primary)` / `var(--demo-accent)` for tenant accents

Do NOT output a full HTML document. The snippet drops into `index.html` after the existing `#publish-schedule` section.
