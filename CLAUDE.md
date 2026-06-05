# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A single-page, zero-build AI token usage dashboard for a fictional 100-person company ("Meridian Systems"). It surfaces the real "token maxing" phenomenon — employees burning through 5-hour rolling Claude windows, ballooning per-seat costs, and CFO-level spend risk — through fake live data.

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
- Feed entries: every 1.1s — picks a random employee, appends a fake action to the activity log
- Token counter: every 1s — increments `totalTokens` by 18K–72K, updates KPI card and last chart data point
- Quota events: every 9s — randomly maxes out a heavy user (top 35), auto-unblocks after 20–45s
- Window saturation grid: every 3s — nudges each employee's `windowUsed` upward, turns cell red when full

**Chart:** Chart.js line chart with dual Y axes (tokens left, spend right), 30-day series with only the first 17 days populated (simulating mid-month).

## Key Design Decisions

- The 5-hr rolling window and per-tier token caps (Pro: 44K, Max5: 88K, Max20: 220K) match Anthropic's actual May 2026 limits — this grounds the fiction in real numbers.
- Department spend averages are calibrated so Engineering (~$1,640 avg) dominates, reflecting real-world Claude Code usage patterns ($500–$2,000/engineer/month).
- "Token maxing" framing: the quota alerts, saturation heatmap, and "Frequent Maxers" panel are the editorial core — they make the CFO case visible at a glance.
