# Landing-page demo roadmap

Nav links 6 anchors. Each should land on an interactive demo section like `#publish-schedule` already does. Built into `index.html` as static + JS-driven interactions (no backend).

| Anchor | Feature | Status |
|---|---|---|
| `#schedules` | Schedule publisher | done — `#publish-schedule` accordion-item |
| `#registrations` | Parent signup flow | designed (2026-04-29) — see `prompts/registrations.md`. Pending: build into index.html |
| `#teams` | Roster + team builder | designed (2026-04-29) — see `prompts/teams.md`. Pending: build into index.html |
| `#scores` | Live scoring + ticker | designed (2026-04-29) — see `prompts/scores.md`. Pending: build into index.html |
| `#standings` | Standings table | designed (2026-04-29) — see `prompts/standings.md`. Pending: build into index.html |
| `#notifications` | Parent broadcast/update | designed (2026-04-29) — see `prompts/notifications.md`. Pending: build into index.html |

## Design checkpoint — 2026-04-29

All 5 outstanding nav-anchor sections + 4 additional surfaces have approved design briefs in `prompts/`. Each prompt is self-contained for paste into a fresh design agent (Claude Code cloud / Cloud Design / similar). Shared style canon lives in `prompts/_style-guide.md`.

**Nav-anchor demos (new sections to add):**
- `prompts/registrations.md` — 3-step parent-signup wizard
- `prompts/teams.md` — drag/tap roster builder + auto-balance
- `prompts/scores.md` — live scoring + ticker
- `prompts/standings.md` — sortable division table + last-3 sparkline
- `prompts/notifications.md` — segment broadcast composer

**Additional surfaces (revamps + 1 new):**
- `prompts/coach-booking.md` — Calendly-style booking page (Coach $24 tier proof) — NEW section `#coach-booking`
- `prompts/family-calendar.md` — revamps existing `#surfaces` (interactive calendar + family update pane)
- `prompts/league-site.md` — revamps existing `#platform` (interactive mini-site browser frame, 5 tabs)
- `prompts/marketplace.md` — revamps existing `#marketplace` (auto-typing demo + 3 audience facet sets)

Next up: implementation. Build order recommendation in the section below remains valid (cheapest first: Standings, Scores, Notifications, Registrations, Teams). Revamps are larger and benefit from being last so style tokens are settled.

---

## Build order (cheapest → priciest)

### 1. `#standings` — Standings table (~half day)

Mostly already there per `#standings-demo` div. Surface as a top-level section.

- **What to show:** sortable W/L/Pct/GB table for one division, ~6 teams. Animated sort on column click.
- **Sample data:** Bayside teams already used elsewhere (Firecrackers, Panthers, Sparks, Pitch — extend to 6)
- **Interactions:** click column header to sort; click team name to highlight row + show last-3-results sparkline
- **Component:** `<table>` with `aria-sort` + small JS sorter. ~80 LoC.

### 2. `#scores` — Live scoring + ticker (~1 day)

Reuse existing `score-hero` + `ticker-track` markup. Wrap as standalone demo.

- **What to show:** 1 live game card with click-to-advance state (top 5 → bottom 5 → final). Ticker auto-scrolls 4 nearby games.
- **Interactions:** "+1" buttons next to each team to bump score; "Next inning" button advances game state; final state shows W/L recap card
- **Sample data:** Bayside Firecrackers vs Panthers, plus 4 ticker games
- **Effort:** ~150 LoC, mostly state management for inning/score in JS object

### 3. `#registrations` — Parent signup flow (~1.5 days)

3-step wizard for parent registering a kid. Most-common landing surface use case.

- **Step 1:** Pick event from dropdown (3 sample events with prices)
- **Step 2:** Player info (first name, age, jersey size) + auto-fill demo
- **Step 3:** Stripe-styled checkout summary with sticker / service fee / total breakdown
- **Interactions:** form validation, Back/Next navigation, "Pay $X" button → success modal "Thanks, see you Saturday!"
- **No real Stripe** — fake checkout. Just shows the flow.
- **Effort:** ~250 LoC HTML/JS, careful state machine

### 4. `#teams` — Roster + team builder (~1.5 days)

Drag-to-assign players into teams. Showcases roster management.

- **What to show:** unassigned players column on left, 3 teams on right (Panthers/Firecrackers/Sparks), drag pills between
- **Interactions:** drag player from unassigned → team; auto-balance button distributes evenly; click team name to rename inline; click player pill to expand mini-card with age/skill
- **Data:** 12 sample players with pre-set names + jersey numbers
- **Effort:** ~200 LoC + drag-drop polyfill or HTML5 drag API. Mobile fallback: tap-to-select + tap-team-to-assign

### 5. `#notifications` — Parent broadcast/update (~1 day)

Mock the Club tier broadcast email tool from the pricing page.

- **What to show:** segment picker (U10 fall, U12 spring, all parents) → message composer with template variables → preview pane that swaps {first_name} → "Send" → success toast with delivery count
- **Interactions:** type in composer, watch preview pane update live; switch segments and watch recipient count change; "Schedule for later" toggle adds datepicker
- **Sample data:** 3 segments with hardcoded recipient counts
- **Effort:** ~180 LoC, mostly text-input plumbing + live preview

---

## Cross-cutting work (do once, applies to all 5)

- **Section template:** wrap each in same `<section class="demo-section" id="...">` with eyebrow + h2 + intro copy + interactive area + "Try it on your own" CTA → `play.leag.app/account/sign-up`
- **CSS:** reuse existing tokens (`--surface`, `--green-hi`, etc.) — no new design system needed
- **Sample data:** consolidate Bayside teams + players into one `const SAMPLE = {...}` block at top of `<script>` so all demos share it
- **Section order in DOM:** match nav order (Registrations → Teams → Schedules → Scores → Standings → Notifications) so anchor scroll feels natural
- **Mobile:** every demo needs a mobile fallback. Drag-drop in #teams is the trickiest — others mostly work as-is

## Effort total

~5-6 dev-days solo to ship all 5. Standings is the cheapest first win (can ship same day). Notifications is the highest-conversion demo because it maps to the Club $39 tier value prop on pricing page.

## Sequencing recommendation

1. **#standings** — half day; easy win, surfaces existing partial work
2. **#scores** — 1 day; reuses ticker chrome; visually arresting
3. **#registrations** — 1.5 days; most-common visitor use case
4. **#notifications** — 1 day; ties to Club-tier pricing pitch
5. **#teams** — 1.5 days; drag-drop is fiddly; do last when you've got rhythm

If pressed for time, ship 3 (standings + scores + registrations) and leave teams + notifications as "Coming soon" with a placeholder card linked from nav. Pricing page already discloses roadmap-as-it-ships.
