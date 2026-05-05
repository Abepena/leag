# Retrospective: LEAG Landing Page

## Milestone: v1.0 — Step 4 Scoreboard + Standings Swap

**Shipped:** 2026-05-05
**Phases:** 2 | **Plans:** 6 | **Tasks:** 10

### What Was Built

Step 4 of the playground accordion now hosts a polished scoreboard widget + dynamic standings card lifted from `_preview/organize.html`. Hero scoreboard mirrors live state (6 IDs). Standings respond to game outcomes (W/L bump on End game, pct recompute) and to playground sport-chip selection (softball/basketball/soccer/pickleball). 2-column layout on desktop ≥1024px, 1-column on mobile. Zero console errors, zero duplicate IDs across 183 IDs.

### What Worked

- **Single-file scope discipline.** Every plan modified only `index.html`. Made commits trivial to review, made cross-IIFE contracts the only abstraction concern.
- **IIFE pattern for embedded demos.** Scoreboard + standings live as scoped IIFEs under `#run-scoreboard` / `#run-standings` with `root.querySelectorAll(...)` everywhere. Zero global namespace pollution. Cross-IIFE communication via `window.renderRunStandings` / `window.applyFinalToStandings` — clean contracts, easy to grep.
- **Atomic commits per task.** 16 commits across milestone, every code change traceable. Commit messages follow `feat(phase-plan-NN): ...` / `fix(scope): ...` conventions. PR reviewers can read top-to-bottom.
- **Inline browser verification via claude-in-chrome.** When the org usage limit hit mid-Phase 1 Plan 02 (executor agent killed before SUMMARY), the orchestrator picked up Task 2 (human-verify checkpoint) inline via browser automation instead of re-spawning a fresh agent for verification only. Saved tokens, kept momentum.
- **Skip-discuss workflow gate.** Pre-set `workflow.skip_discuss=true` for this project — single-file polish doesn't need batch grey-area Q&A. Auto-generated minimal CONTEXT.md from ROADMAP goal + success criteria. Saved a discuss step per phase.
- **Accordion overflow root-cause fix.** When user reported step 4 widgets clipping, traced through 8 levels of DOM to find `.accordion-list { display: grid }` with no `grid-template-columns` defaulting to min-content (1822px wide). Single-line fix: `grid-template-columns: minmax(0, 1fr)`. Cascade resolved.

### What Was Inefficient

- **Org usage limit interrupted mid-milestone.** Plan 02 executor killed by org monthly usage limit after 1 commit (state machine port) but before SUMMARY + checkpoint. Required orchestrator to write SUMMARY inline + handle checkpoint via browser instead of subagent. In future, watch token burn earlier and stage subagent invocations more carefully when approaching limit.
- **Skipped formal verifier subagent.** No `gsd-verifier`-produced VERIFICATION.md files (0/2 phases). Audit had to manually compose evidence chain from SUMMARY frontmatter + commit history + inline browser sweep. Worked, but next milestone should run verifier per phase to keep audit artifacts self-contained.
- **Skipped plan-checker subagent for Phase 2.** Saved tokens but lost the second-pair-of-eyes gate. Acceptable here because plans were small and well-scoped, but bigger phases should run the checker.
- **Initial scoreboard CSS overflowed silently.** The clipping bug was present from Phase 1 Plan 01 (HTML lift) — scoreboard overflowed accordion at desktop widths. User caught it after milestone "complete". The CSS lifted from organize.html assumed a wider container; needed media-query adaptation that wasn't in the original plan acceptance criteria.
- **Standings basketball title showed wrong team names statically.** Static markup had `<h2>12U basketball standings</h2>` hardcoded as default; render fn updated tbody but not title. Two-line bug took 4 minutes to fix once spotted. Should have noted in initial Plan 03 that title needs to sync per render.

### Patterns Established

- **Cross-IIFE bridge via `window.*` pattern.** `window.renderRunStandings(sport)` (Plan 1.3) + `window.applyFinalToStandings(home, away)` (Plan 2.1). Future demo embeds should follow the same pattern: each IIFE exposes its render/mutation surface on `window` so other IIFEs can call without sharing scope.
- **Hero binding inside render fn.** Single render fn writes both internal widget DOM + external hero IDs in one pass, on every state change. No subscription/listener machinery needed. Pattern: keep DOM writes co-located with state changes.
- **`minmax(0, 1fr)` everywhere a grid hosts overflow-prone content.** Default `1fr` resolves to `minmax(auto, 1fr)` which honors min-content. For grids hosting tables, lists with long content, or nested grids, always use `minmax(0, 1fr)` to allow shrinking below intrinsic content width.
- **Sport-keyed data structure with `ours` flag.** Each sport entry has `ours: "TeamName"` + rows array; render fn marks the row matching ours with `ours-row` class. Easy to extend to new sports — just add a top-level entry.

### Key Lessons

- **Live verification > formal verification when surface is small + visual.** For a single-file landing-page polish, claude-in-chrome browser sweep produces stronger evidence than a verifier subagent reading file diffs. Use the verifier when the surface is API contracts, multi-file refactors, or non-visible state.
- **Accordion + nested grids = overflow trap.** Any time content grids (tables, multi-col layouts) live inside an accordion or collapsible container, the parent grid needs explicit column constraints. Default behavior almost always overflows.
- **Sync static fallback markup with render fn output.** When a render fn replaces part of the DOM on boot, the static fallback markup should match what the render fn would write (not a different sport's title, not a placeholder). Otherwise pre-script-load flash + post-script glitches.
- **Tech debt is process-level when verification is inline.** "Missing VERIFICATION.md" doesn't mean "unverified" — it means "no formal artifact". Audit status `tech_debt` (not `gaps_found`) is the right call when the chain of evidence holds.

### Cost Observations

- **Model mix:** 100% Opus (per CLAUDE.md `Always inherit the parent model on Agent calls`). Max plan = sunk cost.
- **Sessions:** 1 (this session, with org usage limit hit + recovery + lifecycle)
- **Notable:** Org monthly usage limit hit mid-Plan 02. Orchestrator handled inline → no quality regression, just shifted work from subagent to direct. Suggests orchestrator can effectively substitute for executor on small/visual tasks if needed.

## Cross-Milestone Trends

(First milestone — trends will populate from v1.1 onward.)
