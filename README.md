# APS / PCS Scheduler

A web-based scheduling tool for planning Acute Pain Service (APS) and Periop Consult Service (PCS) rotations.

## What it does

- Generates the rotation calendar for a date range: APS runs Friday–Thursday with post-call the following Friday; PCS runs Saturday–Friday with post-call the following Monday.
- Automatically shifts a post-call day off a weekend or a configured holiday (New Year's, MLK Day, Memorial Day, July 4th, Labor Day, Thanksgiving, Christmas, or any custom date).
- Tracks each person's eligibility (APS, PCS, or both), per-week availability (requested / unavailable), and a **separate target week count per service**.
- Imports the real per-person request spreadsheets people send back (free-text "Yes"/"No"/"vac"/etc. answers), with a review screen before anything saves.
- Runs a randomized search (greedy construction + simulated annealing, multiple restarts) to generate a schedule that maximizes coverage, then honored requests, then closeness to each person's target week counts — while respecting a configurable minimum rest period and no-back-to-back-same-service rule.
- Supports manually pinning/locking any week to a specific person (for carryover from a prior block or a manual override); the solver always leaves pinned weeks alone.
- Exports the schedule and a per-person Requested-vs-Given rollup to `.xlsx`, in a layout close to the MASTER/FINAL sheets used previously.

## Running it

This is a single self-contained HTML file (`index.html`) — no build step, no server required. Open it directly in a browser, or serve the repo with GitHub Pages / any static host.

It's built to run inside a [Claude Artifact](https://claude.ai) for its data storage and file-save capabilities (`window.claude.use(...)`); outside that environment it falls back to browser `localStorage` for persistence, and file downloads/exports are unavailable.

## Structure

- `index.html` — the entire application (markup, styles, and logic in one file).
