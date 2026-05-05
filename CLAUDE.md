# CLAUDE.md — LEAG Landing Page

This repository hosts the public landing page for **leag.app**, served via GitHub Pages.

## Scope

This repo is **landing-page only**. No application code, no business docs, no planning materials.

All product source code, planning, research, and business materials live in a separate private repo.

## Contents

- `index.html` — single-page landing site
- `CNAME` — GitHub Pages custom domain (`leag.app`)
- `.github/workflows/static.yml` — GitHub Pages deploy workflow
- `favicon.*`, `og-image.*`, `*.svg`, `*.jpg`, `*.png` — static assets

## Deploying

Push to `main`. GitHub Actions deploys the repo root to GitHub Pages. DNS points `leag.app` at `Abepena.github.io`.

## Health Stack

- Lint: none (static HTML). Use `npx html-validate index.html` ad-hoc if needed.
- Test: open `index.html` locally or use `python3 -m http.server` to preview.
- Deploy: push to `main`.

## License

Proprietary. All rights reserved.

<!-- GSD:project-start source:PROJECT.md -->
## Project

**LEAG Landing Page**

Marketing landing page for **leag.app** — multi-tenant SaaS for youth-sports orgs. Single static HTML deployed via GitHub Pages. Brand surface that converts coaches and clubs to platform signups.

Active scope of this GSD milestone: **swap the Step 4 of "Try a sample season" accordion to use the polished scoreboard + standings widgets from `_preview/organize.html` overview tab, replacing the current basic scorekeeper.**

**Core Value:** Step 4 ("Run the season") is the demo's payoff — visitors who reach it should feel the product. The current scorekeeper looks crude vs. the polished widgets in `_preview/organize.html`. Lifting those widgets in raises perceived product quality without changing the surrounding flow.

Hero scoreboard binding (already wired) must keep working. Standings must respond to "Mark final" by bumping W or L for the user's team.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:STACK.md -->
## Technology Stack

Technology stack not yet documented. Will populate after codebase mapping or first phase.
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->

<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
