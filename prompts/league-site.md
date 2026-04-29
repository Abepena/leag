Codebase (current state): https://github.com/Abepena/leag

Please read and address the following:

---

# Demo artifact prompt — Mini league site (`#platform`)

You are revamping the existing `#platform` section ("Your league's home on the web") into an interactive mini-site demo. Currently it shows a static screenshot-style card with a tryouts hero on the left and a 3-step list (Publish the season / Signups into rosters / Keep it current) on the right.

The revamp keeps the headline + 3-step explanation but turns the screenshot into a clickable mini-site frame: tabs (Home / Teams / Schedule / News / About) that swap content live. Demonstrates what a Bayside-Diamonds-branded LEAG-powered site actually looks like to visit.

## Goal

A 16:10 framed "browser window" wrapping a tenant site. Tab nav bottom (Home / Teams / Schedule / News / About). Each tab swaps the panel contents to a different layout:
- **Home:** hero matchup card (current default) with tryouts banner.
- **Teams:** 3 team cards (Diamonds 16U, Diamonds 14U, Diamonds 12U).
- **Schedule:** condensed list of next 5 events.
- **News:** 3 league posts (most recent first).
- **About:** mission statement + contact + social links.

The 3-step "Publish / Signups / Keep current" list stays on the right. Each numbered step gets a "Show this in the demo" link that auto-clicks the relevant tab to demonstrate the step.

## Brand voice

- Direct, technical, no corporate jargon. Avoid "leverage", "utilize", "comprehensive", "robust".
- Founder voice. "We" for LEAG, "you" for the org running the site.
- No emojis. No em-dashes (`—`) or en-dashes (`–`). Hyphens fine.
- Sentence case headings.
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

The mini-site is the tenant's Bayside Diamonds page. Every accent inside the frame uses `var(--demo-primary)` / `var(--demo-accent)`. The frame chrome (URL bar, browser dots, tab nav highlight) uses neutral `var(--ink)` / `var(--quiet)`. The "01 / 02 / 03" step numbers in the right column use `var(--green)` (LEAG brand).

## Section structure

```html
<section class="section" id="platform" aria-labelledby="platform-title">
  <div class="container">
    <div class="section-head">
      <div class="section-kicker">Site</div>
      <h2 id="platform-title">Your league's home on the web.</h2>
      <p>Schedule, teams, standings, tryouts, news, and registration. Wired to live league data.</p>
    </div>
    <div class="ls-grid">
      <div class="ls-frame"><!-- mini-site browser frame --></div>
      <ol class="ls-steps"><!-- 01 / 02 / 03 numbered with deep-links --></ol>
    </div>
  </div>
</section>
```

The `ls-frame` is a stylized browser window:
1. Top chrome row: 3 dots (window controls), URL chip showing `bayside.leag.app`, small "Powered by LEAG" attribution.
2. Tenant header: bracket logo + "Bayside Diamonds" wordmark, tenant tagline.
3. Tenant hero / panel area (changes per tab).
4. Tab nav at the bottom of the panel: Home / Teams / Schedule / News / About.

## Sample data canon

**Tenant:** Bayside Diamonds (Bayside County Sports), `bayside.leag.app`, "Fastpitch / Bayside, FL".
**Brand assets:** `brand/bayside-diamonds-icon.png`, `brand/leag-icon-green.svg`, `brand/leag-wordmark-text-green.svg`.

### Home tab (default)

Hero card content (matches current screenshot):
- Background: `radial-gradient(circle at 30% 30%, color-mix(in srgb, var(--demo-primary) 28%, transparent), transparent 70%)` over a dark surface, with a subtle field-marking overlay (the diagonal lines suggesting infield).
- Sport pill: `Fastpitch / Bayside, FL` (mono caps, `var(--surface-3)` bg).
- Title: `Diamonds 16U open tryouts` (display, italic, uppercase, white).
- Sub-line: `July 8 / Bayside Field 2 / registration required`.
- Lower-right corner: stylized scoreboard glyph or compass rose in `var(--demo-primary)` outline (suggesting "navigate the season").

### Teams tab

3 team cards in a vertical stack (or grid on wide screens):
| Team | Age | Coach | Record | Next |
|---|---|---|---|---|
| Diamonds 16U | 16U | Coach Reyes | 12-3 | vs Marlins, Jun 17 |
| Diamonds 14U | 14U | Coach Hill | 9-6 | @ Comets, Jun 18 |
| Diamonds 12U | 12U | Coach Patel | 7-8 | vs Hammerheads, Jun 20 |

Each card: team name (display italic), coach + age (mono), record (large mono), Next-game line.

### Schedule tab

Condensed list of next 5 events (mirrors `#surfaces` matchup data):
- Jun 16 / Diamonds practice / Field 2 / Scheduled
- Jun 17 / Diamonds vs Marlins / Field 1 / **Live**
- Jun 19 / Comets vs Hammerheads / Field 4 / Scheduled
- Jun 24 / Semifinal bracket opens / Main complex / Open
- Jun 27 / Diamonds vs Tide / Field 1 / Scheduled

### News tab

3 posts, most recent first:
- "Tryouts move to Field 2" — Apr 29, 2026 — "Field 1 turf maintenance bumped tryouts. Same time, new field. Bring cleats."
- "Saturday clinic spots open" — Apr 25, 2026 — "Two spots opened in Coach Reyes's pitching velocity clinic on May 3."
- "Spring season schedule live" — Apr 18, 2026 — "Full season schedule for all 3 Diamonds age groups now visible. Filter by team in the calendar tab."

Each post: title (display italic), date (mono caps), 2-line excerpt.

### About tab

- Mission paragraph: 2-3 sentences ("Bayside Diamonds runs Fastpitch programs for ages 8 through 18 across Bayside County. We're a parent-and-coach-run nonprofit. The season runs Mar through Jul plus a Sept-Oct fall ball loop.").
- Contact card: `coach@bayside.leag.app` (use the existing `.copy-email` clipboard pattern, `data-email="coach@bayside.leag.app"`, `data-copy-label="Email copied"`).
- Field locations list (mono): `Field 1 / Bayside Park`, `Field 2 / Bayside Park`, `Cage A / Bayside Park`, `Main complex / Bayside Tournament Grounds`.

## Demo behavior

### Tab switching

- Click any tab in the bottom nav: panel content swaps with a 220ms cross-fade.
- Active tab gets a `var(--demo-primary)` underline 2px tall + ink-color text. Inactive: `var(--quiet)` text, no underline.
- Default: Home tab.

### Right column step deep-linking

- Each numbered step has a small "See it" link below its body copy:
  - **01 Publish the season** → click "See it" → switches mini-site to Home tab.
  - **02 Signups into rosters** → click "See it" → switches to Teams tab.
  - **03 Keep it current** → click "See it" → switches to News tab (showing real-time updates).
- Active step (matching current tab) gets a faint `var(--green-soft)` background pulse.

### Browser-frame URL chip

- Static text `bayside.leag.app/{tab}` where `{tab}` updates per active tab (e.g., `bayside.leag.app/teams`). Mono font.
- Click on URL chip: copy-to-clipboard via the existing `.copy-email`-pattern handler with `data-copy="bayside.leag.app/teams"` (or whichever active). Toast "Link copied".

### Demo footnote (below the stage, not inside the frame)

`Demo only. Nothing saves, sends, or charges. Real tenant sites pull from live league data and update in real time.`

## Accessibility

- Tab nav is `role="tablist"`, each tab `role="tab"` with `aria-selected` and `aria-controls` linked to the panel.
- Panel has `role="tabpanel"` + `tabindex="0"` so keyboard users can land in it.
- Right-column "See it" buttons are `<button type="button">` elements (not styled `<a>`).
- Frame chrome is decorative (`aria-hidden="true"`).

## Mobile (360px+)

- The browser frame keeps full width but reduces internal padding.
- Right-column steps move below the frame (column flow).
- Tab nav stays sticky at the bottom of the frame (horizontal scroll if 5 tabs don't fit).

## Acceptance criteria

- [ ] 5 tabs each render distinct content per spec.
- [ ] Click any tab swaps the panel with cross-fade.
- [ ] Right-column step "See it" buttons deep-link to correct tabs.
- [ ] URL chip updates per tab and copies on click with toast feedback.
- [ ] Bayside Diamonds wordmark + bracket logo visible in tenant header.
- [ ] About tab uses `.copy-email` pattern for the coach email.
- [ ] Theme picker re-tints all in-frame accents (hero gradient, tab underline, About-page links, post-title accent).
- [ ] No external deps. No console errors. Renders at 360px and 1180px.

## Output format

Return a single self-contained section snippet:
1. The `<section class="section" id="platform">` wrapper (preserve the existing `id="platform"` and `aria-labelledby="platform-title"`).
2. Inline `<style>` block scoped to classes prefixed `ls-` to avoid leaking.
3. Inline `<script>` for tab switching + step deep-links + URL-chip copy (vanilla JS, no deps; reuse the existing global `.copy-email` delegate where possible by adding `class="copy-email"` and a `data-email` attribute, or extend the delegate to also handle a generic `data-copy` payload).
4. Uses `var(--demo-primary)` / `var(--demo-accent)` for tenant accents. Zero use of `var(--red)`.

Do NOT output a full HTML document. This snippet replaces the existing `#platform` section in `index.html`.
