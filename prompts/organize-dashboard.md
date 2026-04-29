Codebase (current state): https://github.com/Abepena/leag

Please read and address the following:

---

# Demo artifact prompt — Organize dashboard (post-login `/organize`)

> **Scope:** Standalone full-page product mock of the LEAG **operator** dashboard org admins land on after login. Output is a single self-contained `.html` file (not an embedded landing-page section). Styling MUST match the existing landing-page demo components (`#coach-booking`, `#registrations`, `#publish-schedule`, etc.) — same dark theme, same tokens, same fonts, same chrome density. Treat the existing demo sections as the design system; this dashboard is just another surface inside that system.

## Goal

A polished, screenshottable mock of `app.leag.app/organize` showing how an org admin works the platform after they log in. Two big design ideas to land:

1. **Simple side tabs** (left rail). Top-level surfaces only — Overview, Schedule, Roster, Registrations, Booking, Comms, Reports, Settings. No tier groupings in the sidebar. Inspiration: the LEAG IG mock saved as `~/Downloads/ChatGPT Image Apr 28, 2026, 05_36_36 PM.png` — tight vertical column, icon + label, brand-green accent on active tab. Reproduce that vibe in the standalone mock.
2. **Lock state lives on the component, not the tab.** Inside each tab, render every feature card the org will eventually see. Cards for features the current tier already owns are fully Live with real-looking data. Cards for higher-tier features render the same data, with a card-level lock chrome (subtle dashed border, "Locked — Club" pill in the top-right corner, dimmed primary CTA, faint upgrade banner across the card footer). The card is still clickable and interactive; writes return a "Demo only — upgrade to <Tier>" toast. The point: organizers play through every feature before paying.

## Hard styling rule

Match `prompts/_style-guide.md` exactly. Do NOT introduce a new palette, new fonts, new corner radii, or a new shadow language. Reuse what `index.html` already uses for `#coach-booking` and `#registrations`. If a token doesn't exist there, don't invent one.

## Brand voice

- Direct, technical, no corporate jargon. Avoid "leverage", "utilize", "comprehensive", "robust", "seamless", "delve".
- Founder voice. "We" for LEAG, "you" for the org admin.
- No emojis. No em-dashes (`—`) or en-dashes (`–`). Hyphens fine.
- Sentence case headings.
- No competitor names (TeamSnap, GameChanger, SportsEngine, Calendly, etc.).
- Numbers as digits.

## Style tokens (declare these on `:root` in the standalone file — values copied from landing-page `_style-guide.md`)

```css
--bg: #0c1616;
--bg-2: #08100f;
--surface: #101918;
--surface-2: #141e1c;
--surface-3: #1b2523;
--ink: #f4f5f4;
--muted: rgba(244, 245, 244, 0.72);
--quiet: rgba(244, 245, 244, 0.48);
--faint: rgba(244, 245, 244, 0.16);
--line: rgba(244, 245, 244, 0.12);
--line-strong: rgba(244, 245, 244, 0.24);
--green: #62b77c;
--green-hi: #7dca96;
--green-soft: rgba(98, 183, 124, 0.14);
--gold: #c9a24b;
--demo-primary: #3667B5;
--demo-accent: #70ECF0;
--radius: 8px;
--font-sans: "Geist", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
--font-mono: "JetBrains Mono", ui-monospace, SFMono-Regular, Menlo, monospace;
--font-display: "Anton", "Geist", sans-serif;
```

Operator chrome (sidebar, top bar, page background, all card frames) uses the neutral LEAG palette only — `--bg`, `--surface`, `--ink`, `--green` for primary actions and active tab accent. Tenant-themed elements (anything that would render on the org's public-facing site — sample team chips, public event cards previewed inside Communications, public org page preview thumbnails) use `var(--demo-primary)` / `var(--demo-accent)` so the demo stays consistent with the landing-page playground theming canon.

## Layout

Full-viewport, 1440×900 minimum. Two regions:

```
┌──────────────────────────────────────────────────────────────────────┐
│ TOP BAR (56px)                                                       │
│ LEAG wordmark · org switcher · current tier pill · upgrade · avatar  │
├────────────┬─────────────────────────────────────────────────────────┤
│            │                                                         │
│  SIDEBAR   │  TAB CONTENT                                            │
│  (220px)   │  (flex)                                                 │
│            │                                                         │
│            │                                                         │
└────────────┴─────────────────────────────────────────────────────────┘
```

### Top bar

- Background: `var(--bg-2)`, 1px bottom border `var(--line)`, 56px tall.
- Left: LEAG wordmark text, `--font-display` italic uppercase, 22px, color `var(--green)`.
- Center-left, in order: org switcher pill ("Bayside County Sports ▾"), tier pill ("Coach · $24/mo" with a 6px green dot prefix, mono caps).
- Right: ghost "Upgrade" button (border `var(--line-strong)`, hover fills `var(--green-soft)`), notifications bell icon, avatar circle with initials "AP" on `var(--demo-primary)` background.

### Sidebar (the load-bearing IA decision)

Match the ChatGPT-image reference. 220px wide, `var(--bg-2)` background, 1px right border `var(--line)`. Vertical list of tabs, top to bottom, in this order with these labels and lucide-style icons (16px, stroke 1.5):

1. **Overview** (`layout-dashboard`)
2. **Schedule** (`calendar-days`)
3. **Roster** (`users`)
4. **Registrations** (`clipboard-list`)
5. **Booking** (`calendar-clock`) — coach private-lesson booking
6. **Comms** (`mail`) — broadcast email
7. **Reports** (`bar-chart-3`)
8. **Settings** (`settings`)

A thin section divider, then a smaller secondary block at the bottom of the sidebar:
- **Billing** (`credit-card`) with current-tier badge on the right
- **Help** (`life-buoy`)
- **Sign out** (`log-out`)

Tab row anatomy: 14px Geist label, 16px icon, 12px gap, 36px tall, 16px horizontal padding. Hover: `var(--surface)` background. Active: 3px `var(--green)` left bar, `var(--green-soft)` background, `var(--ink)` label, `var(--green)` icon. Inactive label color: `var(--muted)`. Active by default in this mock: **Overview**.

NO tier groupings in the sidebar. NO state badges in the sidebar. Sidebar stays simple. All lock chrome lives inside the right-pane cards.

### Tab content area

Background: `var(--bg)`. Top of each tab: a header row with page title (`--font-display` italic 28px uppercase, color `var(--ink)`) on the left and a contextual filter row on the right (a season dropdown, a date range pill, etc., depending on tab). Then the body grid of feature cards.

## Card lock-state pattern (component-level — this is the whole point)

Every feature card has one of three states. The card structure is identical in all three; only chrome and data realism shift.

### Live (unlocked at current tier)

- Frame: 1px solid `var(--line)`, `var(--surface)` background, `--radius` corners.
- Header: card title in Geist 16px weight 600, optional eyebrow above (`--font-mono`, 11px uppercase, `letter-spacing: 0.18em`, color `var(--quiet)`).
- Body: real-looking data — table, chart, list. Use the sample-data canon below.
- Primary CTA: `var(--green)` background, `var(--ink)` text, weight 600.

### Demo (built but tier-locked)

- Frame: 1px **dashed** `var(--line-strong)`, `var(--surface)` background, `--radius` corners.
- Top-right corner overlay pill: small lock glyph + "LOCKED · CLUB" (or whichever tier unlocks it) — `--font-mono`, 10px caps, `var(--gold)` text on `rgba(201, 162, 75, 0.10)` background, 1px `var(--gold)` border, pill radius.
- Body: same real-looking data as Live. NOT blurred. NOT redacted. The whole point is they can see and click everything.
- Primary CTA: dimmed — `var(--surface-3)` background, `var(--quiet)` text, but still clickable. Click triggers a toast (see Toast pattern) instead of a real action.
- Footer banner inside the card: thin row across the bottom, `var(--surface-2)` background, 1px top border `var(--line)`. Text on the left: "Available on Club. You can play with the demo data — your changes won't save." Button on the right: "Upgrade to Club · $79/mo" (filled `var(--green)`, small).

### Soon (not yet built)

- Frame: 1px solid `var(--faint)`, `var(--surface-2)` background, `--radius` corners. Whole card opacity 0.85.
- Top-right pill: "COMING SOON" — `--font-mono`, 10px caps, `var(--quiet)` text, no border.
- Body: skeleton placeholder shapes (gray bars, no data).
- No CTA. No footer. No interactivity.

### Beta carve-out

- Frame: 1px solid `var(--green)`, `var(--green-soft)` background-tint inside the header strip only.
- Top-right pill: "BETA · UNDER YOUR ACCOUNT" — `--font-mono`, 10px caps, `var(--green)` text on `var(--green-soft)`, 1px `var(--green)` border.
- Behaves like Live (real data, real CTAs).

### Toast pattern (for locked-card writes)

Bottom-center, 80px from bottom edge, `var(--surface-3)` background, 1px `var(--gold)` border, 12px radius, 12px padding, 14px Geist text: "Demo only. Upgrade to <Tier> to use real data." Auto-dismiss 3s. Reuse the existing `role="status"` pattern from `index.html`'s clipboard toast.

## Tier perspective

Render the dashboard ONCE, viewing as a **Coach tier admin** (org: Bayside County Sports). This makes the locked-card pattern do the work — Pickup and Coach features render Live, Club/League/Federation features render Demo, with a couple of specific items rendered Soon and one rendered Beta.

(Skip the dual-variant request from the previous draft. One screen, one tier perspective, polished to portfolio quality, beats two half-finished screens.)

## Sample data canon (reuse from `_style-guide.md` — do not invent)

- **Tenant org:** Bayside County Sports (`bayside.leag.app`)
- **Tenant teams:** Diamonds (softball), Breakers (basketball), Nets (soccer)
- **Opponent pool:** Marlins, Hammerheads, Comets, Tide, Stingrays
- **Sample players (use the roster from `_style-guide.md`):** Bree Walden #3 C, Sage Foster #7 SS, Kinley Brooks #9 1B, Marlee Quinn #11 2B, Reese Tatum #14 3B, Wren Halliday #17 OF, Avery Pike #21 OF, Harlow Vance #24 OF, Juno Reid #32 UTIL, Sloane Beck #44 DP — keep the AVG numbers from the canon.
- **Field naming:** Field 1 / Field 2 (softball), Court A / Court B (basketball), Pitch 1 / Pitch 2 (soccer)
- **Coach name (booking tab):** Coach Maria Reyes — same identity used in `#coach-booking`.

## Tab-by-tab content spec

### Overview (active by default)

Header: "Overview." filter row right side: season dropdown ("Spring 2026 ▾"), week range pill ("Week of May 4-10").

KPI strip (4 cards across, mono numerals, tabular-nums) — all Live:
- **Active registrations** — 124 / +18 vs last week
- **Revenue this month** — $4,820 / 47 transactions
- **Pending waivers** — 6 / 4 nudges sent
- **Upcoming events** — 12 / next: Sat May 4

Two-column body grid, 60/40 split:

Left column:
- **Upcoming games** card — Live. List of 5 upcoming games with team chip, opponent, day/time, venue. Sample rows:
  - Diamonds vs Hammerheads — Sat May 4, 9:00 AM, Field 1
  - Breakers vs Stingrays — Sat May 4, 11:00 AM, Court A
  - Nets vs Marlins — Sun May 5, 1:00 PM, Pitch 2
  - Diamonds vs Comets — Sat May 11, 9:00 AM, Field 1
  - Breakers vs Tide — Sun May 12, 12:00 PM, Court B

- **Pickup payment cap** card — Soon. Skeleton showing "$X of $1,000 used this month" placeholder (built but tier doesn't apply at Coach — render Soon for now since the cap-monitoring view itself ships in a later stage).

Right column:
- **Standings** card — Demo (locked, Club). Mini standings table with W-L-Win%, columns: Team / W / L / Pct. Show 5 rows from the opponent pool: Hammerheads 7-1 .875, Sharks 6-2 .750, Marlins 5-3 .625, Tide 4-4 .500, Stingrays 2-6 .250.

- **Reports preview** card — Demo (locked, League). Tiny 4-week revenue line chart, axis labels "Apr 13 / 20 / 27 / May 4", values $980 / $1,210 / $1,340 / $1,290. Card footer: "Available on League. Sample data shown."

### Schedule

Header: "Schedule." filter row: team multi-select pill, week paginator `< Week of May 4-10 >`.

- **Manual schedule (1 team)** card — Live (Coach tier owns this for one team). Day-strip Mon-Sun, with 2 events placed (Sat 9 AM Diamonds practice, Sun 11 AM Diamonds vs Marlins).
- **Schedule generator (basic)** card — Demo (locked, Club). Show a "Generate" wizard preview: form with sport / season length / blackout dates fields, "Run generator" CTA dimmed.
- **Schedule generator (full + divisions)** card — Demo (locked, League). Same generator with extra rows: divisions, standings auto-update, double-header logic. Bracket-shaped preview thumbnail to the right.

### Roster

Header: "Roster." filter row: team dropdown, status filter pill ("All / Pending waiver / Active / Inactive").

- **Roster management (1 team)** card — Live. Player table with the 10 sample players from canon. Columns: # / Name / Pos / Status / Waiver. "Add player" CTA top-right.
- **Roster management (15 teams)** card — Demo (locked, Club). Same table component, but team-tabs across the top: Diamonds / Breakers / Nets / Wolves / Foxes / Comets, with player counts per team. Same 10-player sample on the active Diamonds tab.
- **Bulk roster import** card — Demo (locked, Club). Drop-zone styled area with "Drag CSV here or browse" text, dimmed.
- **Member / parent directory** card — Demo (locked, Club). Search bar + 4 sample contact rows: Hannah Park, Tasha Reid, Lou Vega, Asha Shah, with email + phone + linked-player chips.

### Registrations

Header: "Registrations." filter row: event dropdown ("Spring 2026 Softball — full season ▾"), status filter ("All / Paid / Waitlist / Refunded").

- **Registration list** card — Live. Table of 8 paid registrations matching the registrations.md sample data shape: Player / Event / Sticker / Service fee / Total / Date. Use the three sample events from `prompts/registrations.md`: Diamonds 12U Spring Softball ($245 / $12.75 / $257.75), Diamonds Saturday Skills Clinic ($35 / $2.25 / $37.25), Bayside Spring Pickup League ($15 / $1.70 / $16.70). Mix the rows.
- **Refunds (24-hr window)** card — Live. Sub-list of 1 recent refund, plus a "Window closed" placeholder for older entries.
- **ACH dues + payment plans** card — Demo (locked, Club). 4 KPI tiles (Active plans 47 / Total scheduled $18,420 / Collected this month $6,150 / Failed 2) and a small plan list with 4 family rows: Hartwell, Reyes, Knox, Vega.

### Booking

Header: "Booking." filter row: coach selector ("Maria Reyes ▾"), week paginator.

- **Private-lesson booking** card — **Beta**. This is the in-flight feature this week. Show a mini week grid 4 PM-7 PM weekdays with 4 booked slots, 2 pending, rest open. Selected-slot sticky bar previewed at bottom of the card.
- **Group classes** card — Live. List of 3 group offerings using the canon from `prompts/coach-booking.md`: Pitching velocity clinic / Hitting mechanics small-group / Catcher fundamentals. Show seat counters.
- **Calendar sync** card — Soon. Skeleton placeholder.

### Comms

Header: "Communications." filter row: audience filter pill, sent/draft toggle.

- **Broadcast email** card — Demo (locked, Club). Compose preview: subject line "Spring 2026 — Week 4 update", body preview, segment selector "U10 Diamonds roster · 14 recipients", "Send" CTA dimmed. Below: send-history list of 3 prior sends with open-rate stats (Resend-style).
- **Email templates** card — Demo (locked, Club). 4 saved-template cards.

### Reports

Header: "Reports." filter row: date range, breakdown selector.

- **Basic reports** card — Demo (locked, Club). Revenue + signup count per event, last 6 months. 3-line list: Diamonds 12U Spring Softball $7,140, Saturday Skills Clinic $1,260, Pickup League $480.
- **Advanced reporting** card — Demo (locked, League). Bigger card. Cohort chart placeholder with axis labels, retention column, no-show rate, multi-season comparison toggle, custom date range, "Export CSV" CTA dimmed.
- **Developer API + bulk export** card — Demo (locked, League). "Generate API token" CTA dimmed, sample token chip mocked, 3 endpoints listed: `/api/v1/registrations`, `/api/v1/events`, `/api/v1/players`.
- **Background-check integration** card — Demo (locked, League). 1 sample staff member row with "Request check ($30 at-cost)" CTA dimmed.

### Settings

Header: "Settings."

- **Org profile** card — Live. Org name, contact email, logo upload preview.
- **Custom domain** card — Demo (locked, Club). DNS instructions panel with sample CNAME records targeting `cname.leag.app`, "Verify domain" CTA dimmed.
- **White-label** card — Demo (locked, League). Toggle "Hide LEAG branding from public org page" with preview thumbnails before/after.
- **Staff seats** card — Live (1 of 1 used at Coach tier). Below: a sub-card showing the 5-seat / unlimited-seat upgrade path, locked.
- **Promoted-event placements** card — Demo (locked, League). Shows the 3-included-per-month explainer + a sample featured-listing preview thumbnail.

## Top-bar interaction

- Org-switcher pill: clickable, opens a small popover with 2 sample orgs (Bayside County Sports + Riverstone Athletics).
- Tier pill: hover surfaces a tooltip "Coach tier — $24/mo · 5% + $0.50 service fee".
- Upgrade button: scrolls / links to a `#billing` anchor in the same page that previews the same pricing-card grid from `pricing.html` (in miniature). Link only, no actual route.

## Sidebar interaction

- Clicking a tab swaps the tab content area in place. NO route change. Single page.
- Active tab's left bar transitions in 120ms.
- Keyboard: `↑/↓` cycles tabs when sidebar is focused. `Enter` activates.

## Card interaction

- All Live cards interactive (open mock modals, fill mock forms, etc.) but never persist anything.
- Demo (locked) cards: clicking the dimmed primary CTA shows the toast pattern from above, plus a subtle pulse on the "Upgrade" button in the card footer.
- Soon cards: cursor `not-allowed`, no hover.
- Beta cards: no chrome differences from Live except the green frame + "BETA" pill.

## Reference files (READ before designing)

- `prompts/_style-guide.md` — palette, fonts, sample-data canon. Use these tokens verbatim.
- `prompts/coach-booking.md` — pull the booking-card visual rhythm, slot-grid color states, modal pattern.
- `prompts/registrations.md` — pull the wizard-step look, itemized checkout layout, service-fee breakdown.
- `prompts/standings.md` — pull the standings-table column structure for the Overview standings preview.
- `prompts/scores.md` — pull the linescore tile look if any score tiles appear in Overview.
- `index.html` (deployed at leag.app) — visual ground truth. Match its density, type rhythm, surface stack, button styles. If anything in this prompt conflicts with what's already shipped on leag.app, the shipped version wins.

## Quality bar

- Portfolio-grade screenshot, not a wireframe. Real copy in every space — no Lorem ipsum, no "Description here." If you don't know what a button label should say, write what an organizer would actually click.
- Tabular numerals on every number. Mono font on stat digits, jersey numbers, scores, rates, prices, times.
- The locked-card pattern is the hero moment. Make sure at least 4 different locked cards visibly read as Demo (not Soon, not Live) on the first screen — Roster (15 teams), Standings, Custom domain, ACH dues. The pattern has to be obvious without a legend.
- Sidebar fits all 8 primary tabs + 3 secondary on a 1440×900 viewport without scroll. If it doesn't fit, tighten row height before truncating labels.
- DO NOT design for mobile. Desktop-only at 1440×900 minimum.

## Out of scope

- Mobile responsiveness. Skip it for this mock.
- Real implementations. Every interactive piece is a stub that simulates behavior client-side.
- Multi-tier variants. One screen, Coach perspective, polished.

## Output format

A single self-contained `.html` file:

1. Full HTML document (`<!doctype html>`, `<head>`, `<body>`).
2. Inline `<style>` block — no external CSS, no Tailwind, no Bootstrap. Vanilla CSS using the tokens above.
3. Inline `<script>` for tab switch, mock toast, mock modal interactions. No external deps. No frameworks.
4. Geist + JetBrains Mono + Anton loaded via Google Fonts in `<head>`. Identical font weights and styles to landing page.
5. No README, no extra files, no build step.

Open the result in a browser at 1440×900. Screenshot it. Both the screenshot and the file should feel like a sibling of the existing `#coach-booking` and `#registrations` demos — same dark stage, same green accent, same type rhythm — just expanded into a full operator surface.
