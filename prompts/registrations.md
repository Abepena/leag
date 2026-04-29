# Demo artifact prompt — Registrations (`#registrations`)

You are designing a single interactive demo section for the LEAG landing page (leag.app). This section will be appended to `index.html` and anchored at `#registrations`. It demonstrates the parent signup / event-registration flow without touching a real backend.

## Goal

Show a parent registering a kid for an event in 3 quick steps, ending in a fake checkout summary that breaks down sticker price, service fee, and total. Emphasis: how easy this is for the parent, and how transparent the math is for the org.

## Brand voice

- Direct, technical, no corporate jargon. Avoid "leverage", "utilize", "comprehensive", "robust", "delve", "seamless".
- Founder voice. "We" for LEAG, "you" for the parent.
- No emojis. No em-dashes (`—`) or en-dashes (`–`). Hyphens fine.
- Sentence case headings.
- No competitor names (TeamSnap, GameChanger, etc.).
- Numbers as digits.

## Style tokens (already on `:root` in index.html)

```css
--bg: #0c1616; --bg-2: #08100f;
--surface: #101918; --surface-2: #141e1c; --surface-3: #1b2523;
--ink: #f4f5f4; --muted: rgba(244,245,244,0.72); --quiet: rgba(244,245,244,0.48);
--line: rgba(244,245,244,0.12); --line-strong: rgba(244,245,244,0.24);
--green: #62b77c; --green-hi: #7dca96; --green-soft: rgba(98,183,124,0.14);
--demo-primary: #3667B5;  /* tenant theme primary, picker-driven */
--demo-accent: #70ECF0;   /* tenant theme accent, picker-driven */
--radius: 8px;
--font-sans: "Geist", sans-serif;
--font-mono: "JetBrains Mono", monospace;
--font-display: "Anton", sans-serif;  /* italic uppercase for headlines */
```

Anything tenant-branded (the org's name, primary call-to-action inside the parent flow, success state) uses `var(--demo-primary)` so the playground color picker re-tints it live. Body chrome uses neutral `--surface` / `--ink` / `--line`.

## Section structure

```html
<section class="demo-section" id="registrations">
  <div class="container">
    <div class="section-head">
      <div class="section-kicker">Registrations</div>
      <h2 id="registrations-title">Sign-up that takes 90 seconds.</h2>
      <p>One screen for the parent, one Stripe-clean checkout, no field they have to fill twice.</p>
    </div>
    <div class="demo-stage">
      <!-- 3-step wizard -->
    </div>
  </div>
</section>
```

## Sample data canon

**Tenant org:** Bayside County Sports (`bayside.leag.app`)
**Org teams:** Diamonds (softball), Breakers (basketball), Nets (soccer).
**Opponent pool:** Marlins, Hammerheads, Comets, Tide, Stingrays.

**Three sample events for the picker (Step 1):**

| Event | Sticker | Service fee shown | Total |
|---|---|---|---|
| Diamonds 12U Spring Softball — full season (10 weeks) | $245 | 5% + $0.50 = $12.75 | **$257.75** |
| Diamonds Saturday Skills Clinic — single session | $35 | 5% + $0.50 = $2.25 | **$37.25** |
| Bayside Spring Pickup League — drop-in week | $15 | 8% + $0.50 = $1.70 | **$16.70** |

(The Pickup League event uses the 8% Pickup-tier rate — different rate per event surfaces transparency.)

**Sample player auto-fill for Step 2:** "Avery Pike", DOB 2014-06-12, jersey size YL.

## Demo behavior (3-step wizard)

### Step 1: Pick event

- 3 selectable cards, each showing event name, dates, sticker price, "Service fee 5% + $0.50" subtle line.
- Selected card outlines with `var(--demo-primary)`. Unselected: faint border `var(--line)`.
- "Continue" button (primary, `var(--demo-primary)` background) becomes enabled when one is selected.

### Step 2: Player info

- Form fields: First name, Last name, DOB, Jersey size, Parent email, Parent phone.
- Small "Auto-fill demo data" link in top-right that fills with Avery Pike sample.
- Inline validation (red border + 1-line message) on blur if required field empty.
- "Back" + "Continue" buttons.

### Step 3: Checkout summary

- Itemized breakdown:
  ```
  Event sticker            $245.00
  Service fee (5% + $0.50)  $12.75
  ─────────────────────────────────
  Total                   $257.75
  ```
  (Use spaces for alignment, monospace font; do NOT use em-dashes for the divider — use plain hyphens or a `<hr>`.)
- "Pay $257.75" button (primary).
- Below the button, small footnote: "Demo only. Nothing saves, sends, or charges."
- On click of pay button, swap card content for a success state: large checkmark SVG (no emoji), "Thanks. You're registered." headline, "We sent confirmation to {parent_email}.", "Add to calendar" + "Reset demo" actions.

## Accessibility

- Each step is a `<fieldset>` with `<legend>`. Use `aria-current="step"` on the active step indicator.
- Focus trap inside the active step. "Continue" autofocuses on step transition.
- All form inputs labeled. Errors announced via `role="alert"`.
- Step indicator readable as text by screen readers (`Step 2 of 3, Player info`).

## Mobile (360px+)

- Step indicator collapses to compact pills (`1 / 2 / 3` numerals only with current step labeled below).
- Event cards stack vertically.
- Form fields full-width with `min-height: 44px` tap targets.
- Checkout summary: itemized list stays single column.

## Acceptance criteria

- [ ] All 3 steps reachable + cancellable (Back from step 2 returns to step 1 with selection preserved).
- [ ] Auto-fill demo link works on step 2.
- [ ] Validation messages appear for empty required fields on Continue attempt.
- [ ] Total amount on step 3 reflects the event chosen on step 1 (math correct for all 3 events).
- [ ] Success state replaces card on pay-click; "Reset demo" restarts at step 1.
- [ ] No real network request anywhere. No real Stripe. No console errors.
- [ ] Section uses `var(--demo-primary)` for tenant accents and re-tints when the playground theme picker (already on the page) changes the value via `document.documentElement.style.setProperty`.
- [ ] Renders cleanly at 360px and 1180px container widths.

## Output format

Return a single self-contained section snippet:
1. The `<section class="demo-section" id="registrations">` wrapper
2. Inline `<style>` block scoped to classes prefixed `reg-` to avoid leaking
3. Inline `<script>` for the wizard state machine (vanilla JS, no deps)
4. Uses `var(--demo-primary)` / `var(--demo-accent)` for tenant accents

Do NOT output a full HTML document. The snippet drops into `index.html` after the existing `#publish-schedule` section.
