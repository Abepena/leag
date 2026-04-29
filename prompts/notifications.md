Codebase (current state): https://github.com/Abepena/leag

Please read and address the following:

---

# Demo artifact prompt — Notifications (`#notifications`)

You are designing a single interactive demo section for the LEAG landing page (leag.app), anchored at `#notifications`. Demonstrates the broadcast-message tool: pick a segment, compose a message with template variables, preview, send, see the success toast.

## Goal

Show a coach blasting a season update to a parent segment in 60 seconds. Three pickable segments (with live recipient counts), a composer with `{first_name}` template variable, a side-by-side preview pane that swaps the variable for a sample name, and a send button → toast confirmation with delivery count.

This demo maps directly to Club tier's broadcast email tool on the pricing page — high-conversion section.

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
--green: #62b77c; --green-hi: #7dca96;
--demo-primary: #3667B5;  /* tenant theme primary, picker-driven */
--demo-accent: #70ECF0;   /* tenant theme accent, picker-driven */
--radius: 8px;
--font-sans: "Geist", sans-serif;
--font-mono: "JetBrains Mono", monospace;
--font-display: "Anton", sans-serif;
```

The selected segment, send button, and the preview-pane "from" badge use `var(--demo-primary)`. Template variable `{first_name}` rendering in the composer is highlighted with `var(--demo-accent)` background pill until typed past.

## Section structure

```html
<section class="demo-section" id="notifications">
  <div class="container">
    <div class="section-head">
      <div class="section-kicker">Notifications</div>
      <h2 id="notifications-title">Tell every parent at once. Or just the right ones.</h2>
      <p>Pick a segment, write the message once, watch it personalize. No manual list-pasting. No bcc'ing 80 parents.</p>
    </div>
    <div class="demo-stage">
      <div class="notif-grid">
        <aside class="notif-segments"><!-- 3 segments --></aside>
        <section class="notif-composer"><!-- subject + body + send --></section>
        <section class="notif-preview"><!-- live preview pane --></section>
      </div>
    </div>
  </div>
</section>
```

## Sample data canon

**Tenant org:** Bayside County Sports.
**Three segments:**

| Segment | Recipients | Description |
|---|---|---|
| Diamonds 12U Spring (active roster) | 14 parents | Players currently rostered for the active season |
| All Diamonds families (any season) | 38 parents | All families with any past or current Diamonds enrollment |
| Bayside Spring Pickup League | 22 parents | Drop-in league, current week |

**Sample preview-name (for the live `{first_name}` swap):** the parent of "Avery Pike" is "Jordan Pike", so when a message reads `Hi {first_name}`, preview shows `Hi Jordan`.

**Default composer message (pre-filled, editable):**
```
Subject: Saturday practice moved to Field 2

Hi {first_name},

Quick heads-up. Saturday's practice for Avery's team got bumped from Field 1 to Field 2, same time (10:00 AM). Field 1 is hosting a tournament we couldn't move around.

See you Saturday.

Coach
```

## Demo behavior

### Segment picker (left column)

- Three radio-style cards. Selected gets a 3px left border in `var(--demo-primary)` and primary-tinted name.
- Recipient count number is mono, large.
- Click a different segment: composer message stays, preview pane updates the recipient name to a sample from that segment, send button updates count text.

### Composer (middle)

- Subject input (single line, max 80 chars, char counter under it).
- Body textarea (8 rows default, autoresize on input).
- Inline rendering: any `{first_name}` token gets a `var(--demo-accent)` background pill in the composer. Helper text under body: "Use `{first_name}` to personalize. Other tokens: `{team_name}`, `{next_game_date}`."
- Below body: "Send via" radio: Email (default) / SMS (uppercase, all caps notice "limited preview" — keep email selected by default).
- "Schedule for later" toggle. When on, shows a `<input type="datetime-local">`. When off, send button reads "Send now". When on, send button reads "Schedule send".
- Primary "Send now" button at the bottom in `var(--demo-primary)`.

### Preview pane (right)

- Mock email-client header: "From: Bayside Diamonds <coach@bayside.leag.app>", "To: {sample_first_name} {sample_last_name}", "Subject: ..."
- Live body preview rendering the message with `{first_name}` swapped for the sample name (Jordan Pike for active roster, Riley Quinn for all families, Sam Beck for pickup league).
- Updates on every keystroke in the composer.
- Below preview, a faint footer: "Preview shows what {sample_name} would receive."

### Send action

- Click send: button enters loading state ("Sending..." with a small spinner). After 800ms simulate completion.
- Show a toast at the bottom of the stage: "Sent to 14 parents." in `var(--demo-primary)` background, white text, 4-second auto-dismiss.
- Optionally lock the composer briefly (~600ms) and restore for further demo iteration.

### Demo footnote

Below the stage: "Demo only. Nothing saves, sends, or charges. Real sends go through your verified domain via Postmark."

## Accessibility

- Segment picker is a `role="radiogroup"` with three `role="radio"` cards.
- `{first_name}` token in composer flagged via `aria-label="template variable, will be replaced per recipient"`.
- Toast announces via `role="status"`.
- Preview pane is `aria-live="polite"` so screen readers get periodic updates without spamming.

## Mobile (360px+)

- Three columns collapse to a vertical stack: segment picker (compact horizontal scroller of 3 cards) → composer → preview. 
- Preview can be tab-controlled at narrow widths: small "Preview" tab toggle next to "Compose" tab toggle, swap views.
- Send button sticky to bottom.

## Acceptance criteria

- [ ] All 3 segments render with correct recipient counts.
- [ ] Selecting a segment updates preview-pane sample name + send button text.
- [ ] Live preview updates on every keystroke in subject and body.
- [ ] `{first_name}` token rendered as `var(--demo-accent)` pill in composer.
- [ ] Schedule-for-later toggle reveals datetime input and updates send-button text.
- [ ] Send action: loading state → toast → reset for replay.
- [ ] Theme picker on the playground re-tints accents live (selected segment, send button, toast, preview "from" badge).
- [ ] No external deps. No console errors. Renders at 360px and 1180px.

## Output format

Return a single self-contained section snippet:
1. The `<section class="demo-section" id="notifications">` wrapper
2. Inline `<style>` block scoped to classes prefixed `notif-` to avoid leaking
3. Inline `<script>` for segment select + live preview + send simulation (vanilla JS, no deps)
4. Uses `var(--demo-primary)` / `var(--demo-accent)` for tenant accents

Do NOT output a full HTML document. The snippet drops into `index.html` after the existing `#publish-schedule` section.
