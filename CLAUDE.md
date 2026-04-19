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
