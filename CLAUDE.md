# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A single-page, zero-build AI token usage dashboard for a fictional 430,000-employee company ("Meridian Systems"). The detailed panels show a 100-seat sample ("Eng Platform Org"); the spend odometer and header reflect company-wide scale. It surfaces the real "token maxing" phenomenon — employees burning through 5-hour rolling Claude windows, ballooning per-seat costs, and CFO-level spend risk — through fake live data.

Deployed to GitHub Pages via `.github/workflows/deploy.yml` on every push to `main`: https://nickman710.github.io/tokenMax/

## Running It

```bash
# Any static file server works — e.g.:
npx serve .
python3 -m http.server 8080
# Then open http://localhost:8080
```

No build step. No dependencies to install. Everything is CDN-loaded (Tailwind, Chart.js, Google Fonts).

## Architecture

Single file: `index.html` — all markup, styles, and logic in one place.

**Data layer (all in-memory JS):**
- `employees[]` — 100 objects generated at page load, one per fictional employee, each with `dept`, `tier` (Pro/Max5/Max20), `monthlySpend`, `tokensMonth`, `timesMaxedMonth`, `isMaxedNow`, `windowUsed/windowMax`
- `board[]` — leaderboard-sorted copy of `employees`
- Dept spend and token totals derived on the fly

**Live simulation (setInterval loops):**
- Spend odometer: every 100ms — `(Date.now() - Jan 1 2025 epoch) × $430/sec`. Deterministic, identical for all viewers, monotonic. Annual-pace overage vs the $10B budget is a compile-time constant
- Feed entries: every 1.1s — picks a random employee, appends a fake action to the activity log
- Token counter: every 1s — increments `totalTokens` by 18K–72K, updates KPI card and last chart data point
- Page-view token ticker: every 1.2s — header counter starting at ~1.8K–3.2K, +28–72/tick
- Quota events: every 9s — randomly maxes out a heavy user (top 35) via the shared `maxOut(e)` helper, which blocks, counts the hit, and auto-unblocks after 20–45s **resetting `windowUsed`** (without the reset the saturation grid turns permanently red)
- Window saturation grid: every 3s — accrues `windowUsed` proportionally to each tier's cap (`windowMax * 0.02` max per tick) so Pro doesn't fill faster than Max20; full windows route through `maxOut(e)`

**Charts (Chart.js):**
- 30-day trend: dual Y axes (tokens left, spend right), only the most recent 17 days populated — matches the `/17` month-to-date projection math
- Cost Paradox: 12-month — AI-projected savings (dashed) vs last-year baseline vs current = baseline + tripling AI subscriptions
- Productivity Debt: dual axis — headcount falling in three layoff waves vs backlog spiking after each cut

## Key Design Decisions

- The 5-hr rolling window and per-tier token caps (Pro: 44K, Max5: 88K, Max20: 220K) match Anthropic's actual May 2026 limits — this grounds the fiction in real numbers.
- Department spend averages are calibrated so Engineering (~$1,640 avg) dominates, reflecting real-world Claude Code usage patterns ($500–$2,000/engineer/month).
- "Token maxing" framing: the quota alerts, saturation heatmap, and "Frequent Maxers" panel are the editorial core — they make the CFO case visible at a glance.
