# Demo artifact prompt — Marketplace search (`#marketplace`)

You are revamping the existing `#marketplace` section ("Find a game tonight"). Currently it has 3 audience tabs (Players / Organizers / Venues), Sport / Area / When dropdowns, and a list of 3 listings (12U Open Tryout, Fastpitch Pickup Night, Pitching Clinic) with Reserve / Save / Add-to-checkout buttons.

The revamp keeps the same audience-segmented framing but adds simulated typing + filtering animations across more facets (programs, sports, leagues, teams, coach clinics) and produces a smoother, more "live" feel.

## Goal

Players land first by default. A scripted typing demo runs on first scroll-into-view: it auto-fills Sport → "Fastpitch Softball", Area → "Bayside, FL", When → "Tonight", then animates the listing list filtering down from a stub 12-row pool to the matched 4-5. User can take over at any point: clicking a dropdown halts the auto-demo and switches to manual control.

Audience tabs (Players / Organizers / Venues) restructure facets per audience:
- **Players (default):** Sport / Area / When → listings of programs, leagues, clinics, pickup games, tryouts.
- **Organizers:** Sport / Audience age / Format → listings of available coaches and venues to fill program rosters.
- **Venues:** Time block / Sport / Capacity → listings of organizers looking for a field at that time.

## Brand voice

- Direct, technical, no corporate jargon. Avoid "leverage", "utilize", "comprehensive", "robust".
- Founder voice. "We" for LEAG, "you" for the searcher.
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

Audience tabs, the active facet underline, and primary CTAs (Reserve spot, Add to checkout) use the LEAG `--green` (since the marketplace is LEAG-platform-level chrome, not a tenant page). The "spots remaining" digits use `--green` by default; only switch to `--demo-primary` if the listing is from a Diamonds-branded tenant. Listings that originate from a specific tenant get a small `var(--demo-primary)` left-border bar.

## Section structure

```html
<section class="section" id="marketplace" aria-labelledby="marketplace-title">
  <div class="container">
    <div class="section-head">
      <div class="section-kicker">Marketplace</div>
      <h2 id="marketplace-title">Find a game tonight.</h2>
      <p>Players search by sport, area, and time. Organizers post the spots they need filled. Venues turn empty slots into bookings.</p>
    </div>
    <div class="demo-stage mp-stage">
      <header class="mp-stage-head">
        <div class="mp-headline"><!-- audience-driven sub-headline --></div>
        <div class="mp-stage-meta"><!-- "Demo only / Resets on reload" badge --></div>
      </header>
      <nav class="mp-tabs" role="tablist"><!-- 01 Players / 02 Organizers / 03 Venues --></nav>
      <div class="mp-facets"><!-- 3 dropdowns per audience --></div>
      <ul class="mp-list" role="list"><!-- 4-6 listing rows --></ul>
      <footer class="mp-footnote"><!-- demo footnote --></footer>
    </div>
  </div>
</section>
```

## Sample data canon

**Default audience:** Players.
**Default facets:** Sport = Fastpitch Softball, Area = Bayside, FL, When = Tonight.

### Players-tab listing pool (12 stub rows; auto-demo filters to 5)

Show after default filter applied:
| Title | Meta | Spots | Action |
|---|---|---|---|
| 12U open tryout | Bayside / Tonight / Bayside Field 2 / Tryout / Bayside Diamonds | 4 | Reserve spot |
| Fastpitch pickup night | Bayside / Tonight / Helmets provided / Intermediate | 6 | Save |
| Pitching clinic | Bayside / Tonight / Coach-led / 60 min | 8 | Add to checkout |
| 14U open tryout | Bayside / Sat morning / Bayside Field 1 / Tryout / Bayside Diamonds | 6 | Reserve spot |
| Hitting cage drop-in | Bayside / Tonight / Cage A / Self-paced | 12 | Save |

(Other 7 stub rows: 16U travel pre-season, summer skills clinic, Ironheart fall ball signup, etc. They're filtered out by the default Sport=Fastpitch / Area=Bayside / When=Tonight selection.)

### Organizers-tab listing pool (after switch)

Facets: Sport / Audience age / Format.
Default: Fastpitch / 12U / Pickup.
Listings (3-4 rows):
| Title | Meta | Status |
|---|---|---|
| Coach Maria Reyes | Bayside / Pitching specialist / 4.9 rating | Open for clinics |
| Coach Devon Hill | Bayside / Hitting + fielding / 8 yrs experience | 2 open weeks |
| Bayside Field 2 | Tue/Thu nights / Field rental / 2 hr blocks | $80 / 2hr |
| Cage A indoor | All week / Cage rental / 1 hr blocks | $40 / 1hr |

### Venues-tab listing pool (after switch)

Facets: Time block / Sport / Capacity.
Default: Tonight / Fastpitch / 20+.
Listings (3 rows):
| Title | Meta | Looking for |
|---|---|---|
| Bayside Diamonds 12U | Pickup night / 6:00-7:30 PM / 22 players | Cage or field |
| Bayside Diamonds 14U | Tryout / 5:30-7:00 PM / 28 players | Field |
| Coach Reyes private clinic | Lesson block / 4:00-7:30 PM / 8 students | Cage |

## Demo behavior

### Audience tabs

- 3 tabs: `01 Players` / `02 Organizers` / `03 Venues`. Display font, italic, uppercase. Active tab green-underlined; inactive faded `var(--quiet)`.
- Click switches both the facet row and the listings instantly. Halts the auto-demo if running.

### Facet dropdowns

- 3 native-styled `<select>` elements per audience. Use a custom-styled wrapper but keep the `<select>` underneath for accessibility.
- Dropdown labels (mono, uppercase, `var(--quiet)`): `SPORT` / `AREA` / `WHEN` (Players); `SPORT` / `AUDIENCE AGE` / `FORMAT` (Organizers); `TIME BLOCK` / `SPORT` / `CAPACITY` (Venues).
- On dropdown change: list re-filters with a 180ms cross-fade per row (FLIP-style).

### Auto-demo (runs once per scroll-into-view, default audience = Players)

Sequence (uses `IntersectionObserver` so it triggers when the section enters the viewport):
1. Wait 600ms.
2. Type "Fastpitch Softball" into the Sport dropdown's display surface (each character with a 25-40ms random stagger, mono font typing-cursor blink while typing).
3. After the dropdown fills, animate select to that option (visual only; the underlying `<select>` updates instantly).
4. Wait 250ms. Type "Bayside, FL" into Area.
5. Wait 250ms. Type "Tonight" into When.
6. Wait 400ms. The list re-filters: rows that don't match fade and shrink out (height + opacity 220ms), matching rows slide into place.
7. Final state: 5 listings visible.
8. After auto-demo completes, fade in a small caption next to the search bar: `Auto-demo complete. Try changing a filter.`

Auto-demo halts immediately if user clicks any dropdown, tab, or listing button. Captions disappear, manual control takes over.

### Listing rows

- Each row: small square avatar (`SB` for softball, etc., on a `var(--green-soft)` square with `var(--green-hi)` text), title (display, italic, uppercase, white), meta line (mono caps with `·` separators), large mono spots-remaining digit on the right, action button.
- Action buttons:
  - `Reserve spot` (primary green button) → click opens a 1-step confirm modal: "Reserved. We'll hold the spot for 10 minutes while the parent finishes signup." (Demo: just a toast.)
  - `Save` (secondary outlined) → toggles a saved state; toast "Saved to your list".
  - `Add to checkout` (primary) → toast "Added. 1 item in checkout".
- Hover row: `var(--surface-2)` background fill.

### Demo-only badge

Top-right of stage: `Demo only / Resets on reload` (mono caps, faint `var(--quiet)`). Static.

### Demo footnote (below stage)

`Buttons update this page only. No payment or account creation.`

## Accessibility

- Audience tabs are `role="tablist"` with proper `role="tab"` + `aria-selected` + `aria-controls`.
- Facets use real `<label>` + `<select>`. Custom display surface is purely visual; never break native keyboard interaction.
- Listings are `role="list"` with `role="listitem"` rows. Each action button has descriptive `aria-label="Reserve spot for 12U open tryout, 4 spots remaining"`.
- Auto-demo respects `prefers-reduced-motion`: skip typing animation, just instantly populate the facets and re-filter without cross-fade.
- Live region (`role="status"`) for toasts.

## Mobile (360px+)

- Tabs collapse to a horizontal scroll-snap row.
- Facets stack vertically full-width.
- Listing rows reduce to: avatar + title + meta + action button (spots-digit moves to inline next to the title).
- Auto-demo still runs but typing speed accelerates ~30% on mobile.

## Acceptance criteria

- [ ] All 3 audience tabs render distinct facet rows + listing pools per spec.
- [ ] Auto-demo runs on first scroll-into-view with simulated typing into all 3 facets, then list filter animation.
- [ ] User clicking any control halts auto-demo; manual filter changes work.
- [ ] At least 5 Players listings, 4 Organizers listings, 3 Venues listings present.
- [ ] Tenant-origin listings (Bayside Diamonds rows) get a `var(--demo-primary)` left-border bar.
- [ ] All action buttons fire toast feedback via `role="status"` live region.
- [ ] Theme picker re-tints any tenant-origin accent (left bar). LEAG-chrome accents (greens) stay unchanged.
- [ ] `prefers-reduced-motion` skips typing + transitions but still populates correct final state.
- [ ] No external deps. No console errors. Renders at 360px and 1180px.

## Output format

Return a single self-contained section snippet:
1. The `<section class="section" id="marketplace">` wrapper (preserve the existing `id="marketplace"` and `aria-labelledby="marketplace-title"`).
2. Inline `<style>` block scoped to classes prefixed `mp-` to avoid leaking.
3. Inline `<script>` for tab switching + facet selection + auto-demo state machine + IntersectionObserver gating + reduced-motion respect (vanilla JS, no deps).
4. Uses LEAG `--green` for chrome and `var(--demo-primary)` only for tenant-origin listing accents.

Do NOT output a full HTML document. This snippet replaces the existing `#marketplace` section in `index.html`.
