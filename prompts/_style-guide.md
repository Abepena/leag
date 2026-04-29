# LEAG landing-page style guide

Shared style reference for demo-artifact prompts. Each prompt in this folder duplicates the load-bearing parts so it can be pasted standalone — this file is the authoritative source if anything drifts.

## Brand voice

- **Direct, technical, no corporate jargon.** Avoid: "leverage", "utilize", "comprehensive", "robust", "delve", "seamless", "best-in-class".
- **Founder voice.** First-person plural ("we") when LEAG is speaking; second-person ("you") when addressing the user. Never marketing-passive ("organizations are empowered to").
- **No emojis.** Anywhere. Ever.
- **No em-dashes (`—`) or en-dashes (`–`).** Use periods, commas, or rewrite. Hyphens (`-`) fine.
- **Sentence case headings.** Not Title Case. Example: "Run the season from one place." not "Run The Season From One Place."
- **No competitor names** in copy (TeamSnap, GameChanger, SportsEngine, etc.).
- **Numbers as digits** in marketing prose (`5 teams`, not `five teams`).

## Color tokens (CSS custom properties on `:root`)

```css
--bg: #0c1616;            /* page background */
--bg-2: #08100f;          /* deeper background */
--surface: #101918;       /* card surface */
--surface-2: #141e1c;     /* nested surface */
--surface-3: #1b2523;     /* deepest nested */
--ink: #f4f5f4;           /* primary text */
--muted: rgba(244, 245, 244, 0.72);  /* secondary text */
--quiet: rgba(244, 245, 244, 0.48);  /* tertiary text */
--faint: rgba(244, 245, 244, 0.16);  /* dividers */
--line: rgba(244, 245, 244, 0.12);   /* borders */
--line-strong: rgba(244, 245, 244, 0.24);
--green: #62b77c;         /* brand green, primary CTA */
--green-hi: #7dca96;      /* brand green hover */
--green-soft: rgba(98, 183, 124, 0.14);
--gold: #c9a24b;
--paper: #f4efe6;         /* light-mode-style paper accents */
--paper-ink: #161311;
--red: #d83a48;           /* destructive only — do NOT use as default accent */
--blue: #5b8fb7;
--demo-primary: #3667B5;  /* tenant demo theme primary (default navy) */
--demo-accent: #70ECF0;   /* tenant demo theme accent (default mint/teal) */
--container: 1180px;
--radius: 8px;
--shadow: 0 24px 80px rgba(0, 0, 0, 0.36);
```

**Demo-tenant accents:** Anything inside a "what your league sees" demo block should use `--demo-primary` and `--demo-accent` so the playground theme picker re-tints them live. The picker writes to `document.documentElement.style`, so any `var(--demo-primary)` reference updates automatically.

## Fonts (already loaded via Google Fonts in index.html)

```css
--font-sans: "Geist", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
--font-mono: "JetBrains Mono", ui-monospace, SFMono-Regular, Menlo, monospace;
--font-display: "Anton", "Geist", sans-serif;     /* big italic headlines, scoreboards */
--font-serif: "Instrument Serif", Georgia, serif; /* editorial pull-quotes */
```

Hierarchy:
- Hero h1 / scoreboard team names: `--font-display`, italic, uppercase, `clamp(20px, 2.1vw, 28px)` to `clamp(48px, 6vw, 96px)` depending on context
- Body: `--font-sans`, 15-17px
- Stats / metadata / linescore numbers: `--font-mono`, 10-13px
- Section eyebrows: `--font-mono`, 11px, uppercase, `letter-spacing: 0.18em`, color `var(--quiet)`

## Section pattern

```html
<section class="demo-section" id="anchor-id">
  <div class="container">
    <div class="section-head">
      <div class="section-kicker">{Eyebrow}</div>
      <h2 id="anchor-id-title">{Headline}.</h2>
      <p>{One-sentence intro, ≤ 140 chars.}</p>
    </div>
    <div class="demo-stage">
      <!-- interactive demo body -->
    </div>
  </div>
</section>
```

`.demo-stage` is the canonical wrapper for any tenant-facing mock; styles inside should reference `--demo-primary` / `--demo-accent` for accent colors.

## Sample data canon

**Tenant org:** Bayside County Sports
**Tenant slug:** `bayside.leag.app`
**Sports → org teams:** Softball = Diamonds. Basketball = Breakers. Soccer = Nets.
**Opponents (universal pool):** Marlins, Hammerheads, Comets, Tide, Stingrays.
**Brand assets:**
- `brand/bayside-diamonds-icon.png`
- `brand/bayside-breakers-icon.png`
- `brand/bayside-nets-icon.png`
- `brand/hammerheads-icon.png`
- `brand/stingrays-icon.png`

**Hero scoreboard convention:** Diamonds (away/batting, demo-primary tinted) vs Hammerheads (home, neutral). Bases tint with `--demo-accent`.

**Sample players (use for any roster demo, ages within sport-appropriate ranges):**

| # | Name | Pos | AVG / OPS / etc. |
|---|---|---|---|
| 3 | Bree Walden | C | .402 |
| 7 | Sage Foster | SS (captain) | .388 |
| 9 | Kinley Brooks | 1B | .351 |
| 11 | Marlee Quinn | 2B | .342 |
| 14 | Reese Tatum | 3B | .376 |
| 17 | Wren Halliday | OF | .412 |
| 21 | Avery Pike | OF | .298 |
| 24 | Harlow Vance | OF | .331 |
| 32 | Juno Reid | UTIL | .301 |
| 44 | Sloane Beck | DP | .367 |

**Field naming:** "Field 1" / "Field 2" for softball, "Court A" / "Court B" for basketball, "Pitch 1" / "Pitch 2" for soccer.

## Component primitives already in the page

- `.btn`, `.btn-primary` (green), `.btn-secondary` (outlined ink)
- `.demo-stage` wrapper (1px line border, dark surface bg)
- `.tier`, `.tier.featured` (pricing-card style)
- `.matrix` (capability table — yes/no/note classes on cells)
- `.team-chip` + `.team-swatch` (team color dot + name pill)
- `.live-pill` (LIVE NOW badge — `--demo-primary` background)
- `.linescore` (baseball box-score grid)
- `.copy-email` anchor (clipboard email link — `data-email="..."` + `data-copy-label="..."`; global JS handler in index.html copies on click)

## Output format expected from a mockup prompt

A single self-contained HTML snippet:
1. Root `<section class="demo-section" id="...">` matching the anchor
2. Inline `<style>` block scoped to the section's classes (avoid leaking to the page)
3. Inline `<script>` block for interactions, no external deps
4. Uses `var(--demo-primary)` / `var(--demo-accent)` for any tenant-themed accents
5. Mobile responsive at 360px width minimum

Do not output a full HTML document. Just the section snippet that drops into `index.html` after the appropriate sibling section.
