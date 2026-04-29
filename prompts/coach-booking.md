Codebase (current state): https://github.com/Abepena/leag

Please read and address the following:

---

# Demo artifact prompt — Coach booking (`#coach-booking`)

You are designing a single interactive demo section for the LEAG landing page (leag.app), anchored at `#coach-booking`. This is a Calendly-style booking surface for a single coach: post group classes (open enrollment, capped seats) and open private-lesson availability that parents can book directly.

## Goal

Show a coach's public booking page from a parent's perspective. Two stacked offerings:
1. **Group classes** — fixed dates, capped seats, "Reserve spot" CTA per class (multi-spot ok).
2. **Privates** — week-grid of 30-minute slots; parent picks a slot, lands in a 2-step booking modal (player info → checkout summary).

This demo maps to the Coach $24 tier on the pricing page — single coach with single team running their own booking. High-conversion target for solo coaches who currently use Calendly + Venmo.

## Brand voice

- Direct, technical, no corporate jargon. Avoid "leverage", "utilize", "comprehensive", "robust", "seamless".
- Founder voice. "We" for LEAG, "you" for the parent (or "Coach" when speaking about the coach).
- No emojis. No em-dashes (`—`) or en-dashes (`–`). Hyphens fine.
- Sentence case headings.
- No competitor names (Calendly, Acuity, GameChanger, etc.) in copy. Internal context only.
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

Coach's identity (avatar ring, name, primary CTAs, selected slot, confirm button) uses `var(--demo-primary)`. Hover-state on slots and the "spots-remaining" pip use `var(--demo-accent)`. Sold-out slots: `var(--quiet)` text + diagonal-stripe background.

## Section structure

```html
<section class="demo-section" id="coach-booking">
  <div class="container">
    <div class="section-head">
      <div class="section-kicker">Coach booking</div>
      <h2 id="coach-booking-title">Your booking page that takes Venmo out of the chat.</h2>
      <p>Post group classes, open private slots, and let parents book without DM-tagging you. Payment, waiver, and reminder built in.</p>
    </div>
    <div class="demo-stage">
      <div class="cb-grid">
        <aside class="cb-coach-card"><!-- coach identity panel --></aside>
        <section class="cb-offerings"><!-- tabs: Group / Privates --></section>
      </div>
    </div>
  </div>
</section>
```

## Sample data canon

**Coach:** "Coach Maria Reyes" / Bayside Fastpitch / 12-year coaching tenure / "ASA-certified pitching specialist".
**Coach avatar:** circular placeholder with initials "MR" on `var(--demo-primary)` background, white text, with a subtle ring in `var(--demo-accent)`.
**Booking page slug:** `coach.maria.leag.app/book` (visible in a small URL chip below the coach card).

### Group classes (3 offerings, fixed dates)

| Title | When | Where | Price | Cap | Reserved | Status |
|---|---|---|---|---|---|---|
| Pitching velocity clinic | Sat May 3 / 9-11 AM | Bayside Field 2 | $45 | 8 | 6 | 2 spots left |
| Hitting mechanics small-group | Sun May 4 / 10-11:30 AM | Bayside Cage A | $35 | 6 | 6 | Sold out (waitlist open) |
| Catcher fundamentals | Sat May 10 / 9-10:30 AM | Bayside Field 1 | $30 | 10 | 2 | 8 spots left |

Each card shows: title (display font, italic, uppercase), meta line (when / where / price, mono caps), spots-remaining counter (large `var(--demo-accent)` digit), "Reserve spot" / "Join waitlist" / "Reserve spot" CTA.

### Privates (week grid)

- Showed week of May 5-11.
- 30-minute slots between 4:00 PM and 7:30 PM weekdays, 8:00 AM-12:00 PM Saturday.
- Mix of states per slot: open, pending (someone has it in checkout), booked (sold out), blocked (coach blocked the slot).
- Pricing: $40 per 30-min private. (Tier displayed once at top of privates pane.)
- ~12-14 slots open across the week. Mark 4 as booked (Tue 4:30, Wed 5:00, Thu 6:00, Sat 9:00) and 1 as pending (Fri 5:30).

## Demo behavior

### Coach card (left aside)

- Avatar + name + role.
- 1-line bio: "12 years coaching Bayside Fastpitch. ASA-certified pitching specialist."
- Booking-page URL chip (mono): `coach.maria.leag.app/book`.
- Quick stats row (mono digits): `47` lessons booked this season, `4.9` rating, `92%` repeat-rate.
- "Share booking link" button (secondary): copy-to-clipboard the URL chip with toast feedback ("Link copied"), reuse the existing `.copy-email`-style click delegate but `data-copy` instead of `data-email`.

### Offerings panel (right section)

Tab toggle at top: `Group classes` / `Privates`. Default `Group classes`.

#### Group classes view

- Grid of 3 cards as described in canon.
- "Reserve spot" button on cards with spots: opens 2-step booking modal (see below).
- "Join waitlist" on sold-out card: opens 1-step modal asking for parent email + player name + position; Submit → toast "You're on the list. We'll email if a spot opens."
- "Reserve spot" on the third card: same modal as first.

#### Privates view

- Week navigation header: `< Week of May 5-11 >` with prev/next arrows. Static week-of label, both arrows disabled (demo only, doesn't actually navigate).
- 7-column grid (Mon-Sun). Each column = day name + date. Time-slot pills stack vertically inside each column.
- Slot states:
  - **Open:** ink text on `var(--surface-2)` bg, 1px `var(--line)` border, hover `var(--demo-accent)` border, click selects.
  - **Selected:** `var(--demo-primary)` background, white text.
  - **Pending:** dotted border, "pending" mono caption.
  - **Booked:** diagonal-stripe bg, `var(--quiet)` text, `aria-disabled="true"`, no hover.
  - **Blocked:** solid `var(--surface-3)` bg, no text, no hover.
- Below grid, a sticky bottom action bar appears once a slot is selected: "Booking Tue May 6 at 4:30 PM / 30 min / $40" + "Continue to player info" CTA in `var(--demo-primary)`.

### Booking modal (2 steps for both group + privates)

#### Step 1: Player info

- Inputs: First name / Last name / DOB / Position (dropdown: Pitcher / Catcher / Infielder / Outfielder / Other) / Parent email / Parent phone.
- Auto-fill demo link top-right ("Use sample data") fills Avery Pike + DOB 2014-06-12 + Pitcher + parent jordan.pike@example.com + 555-0142.
- "Back" + "Continue" buttons.

#### Step 2: Checkout summary

Itemized:
```
Pitching velocity clinic     $45.00
Service fee (5% + $0.50)      $2.75
─────────────────────────────────────
Total                       $47.75
```
(Use plain `<hr>` or hyphen string for divider, never em-dash.)

- "Pay $47.75" button (primary, `var(--demo-primary)` bg).
- Footnote: "Demo only. Nothing saves, sends, or charges."
- On click: 800ms simulated load → success state inside modal: large checkmark SVG, "You're booked.", "Reminder will go to jordan.pike@example.com 24 hours before.", "Add to calendar" + "Close" actions.

### Demo footnote

Below the stage: "Demo only. Nothing saves, sends, or charges. Real bookings sync to the coach's calendar and trigger reminders 24h before."

## Accessibility

- Tab toggle is `role="tablist"` with `role="tab"` children + `aria-controls` linking to the panel.
- Slot pills are `<button>` with `aria-label="Tuesday May 6, 4:30 PM, 30 minutes, $40, available"` and explicit `aria-disabled="true"` on booked/blocked.
- Modal traps focus inside. Esc closes.
- Auto-fill link is a real `<button>` not a styled `<a>`.
- Copy-link toast uses `role="status"`.

## Mobile (360px+)

- Coach card collapses to a horizontal strip above the offerings (avatar + name + booking URL truncated, tap to expand).
- Group cards stack full-width.
- Privates week grid: 7 columns becomes a horizontal scroll-snap list, one day visible at a time + small day-pager dots underneath. Tap a day, vertical scroll for slots.
- Booking modal goes full-screen on mobile.

## Acceptance criteria

- [ ] All 3 group cards render with correct meta + spot counts.
- [ ] Privates grid renders for 7 days with mixed slot states.
- [ ] Selecting a slot reveals the sticky action bar and is reflected via aria-pressed.
- [ ] Both flows (group + privates) reach the same 2-step modal with itemized math.
- [ ] Pay-button success state inside modal works; close returns to original tab.
- [ ] Auto-fill demo link fills sample player data.
- [ ] Coach card "Share booking link" copies the URL chip with a `role="status"` toast.
- [ ] Theme picker on the playground re-tints coach avatar, primary CTAs, selected slot, success state.
- [ ] No external deps. No console errors. Renders at 360px and 1180px.

## Output format

Return a single self-contained section snippet:
1. The `<section class="demo-section" id="coach-booking">` wrapper.
2. Inline `<style>` block scoped to classes prefixed `cb-` to avoid leaking.
3. Inline `<script>` for tab switch + slot select + modal state machine + clipboard share (vanilla JS, no deps).
4. Uses `var(--demo-primary)` / `var(--demo-accent)` for tenant accents. Zero use of `var(--red)`.

Do NOT output a full HTML document. The snippet drops into `index.html` after the existing `#publish-schedule` section.
