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
- Lets anyone leave a free-text comment on any week regardless of its status (available, requested, or unavailable) — private to that person and to admins.
- Groups everything into **scheduling periods**. Admins can freeze ("close") staff self-service requests without losing anyone's roster entry, and can close out a period entirely — archiving its roster, requests, comments, and schedule as read-only history — while starting the next one with a clean slate for availability (target-week counts and "requested weeks only" carry over). Admins can browse past periods from a selector in the top bar; the Report tab compares the current period's per-person outcomes against the immediately preceding one.

## Accounts & permissions

Sign-in is email/password via Firebase Authentication (project `aps-cps-scheduler`). Two roles:

- **Admin** — can change the rotation window, holidays, and solver settings (Setup tab); add, edit, or remove anyone on the roster; generate the schedule and pin/lock individual weeks; open/close staff requests and close out/start scheduling periods; browse archived periods. Admins are listed by email in the `admins` collection in Firestore. The account at `statzern@gmail.com` self-provisions as the first admin on its first sign-in (see the bootstrap rule in `firestore.rules`); every admin after that is added by an existing admin, either from the console or the in-app admin-management screen.
- **Staff** — sign in, see the published schedule and report (read-only), and edit only their own roster entry's requested/unavailable weeks, target week counts, and per-week comments, under a "My Requests" tab — while the current period has requests open. An admin links a staff account to a roster entry by setting that person's email in their profile; until that's done, the account sees a "not linked yet" message.

Permissions are enforced in `firestore.rules`, not just the UI — the client-side gating is a convenience, the security rules are the real boundary.

## Running it

This is a single self-contained HTML file (`index.html`) — no build step. It needs to be served over HTTP(S) (Firebase Auth doesn't work from a bare `file://` page); either run a static server locally (e.g. `python3 -m http.server` — see `.claude/launch.json`) or use the deployed GitHub Pages site.

## Firebase project setup

The `aps-cps-scheduler` Firebase project backs this app (config is inline in `index.html` — the API key is not secret; access is controlled entirely by `firestore.rules`). One-time setup, already done for this project:

1. Firestore database created, rules deployed: `firebase deploy --only firestore:rules`.
2. A web app registered: `firebase apps:create WEB "APS PCS Scheduler" --project aps-cps-scheduler`.
3. Firebase Authentication → Email/Password sign-in method enabled in the console. **Note:** as of when this was set up, Google requires the project to be on the Blaze (pay-as-you-go) plan before Authentication can be initialized at all, even though normal usage at this scale stays within the always-free tier.

## Structure

- `index.html` — the entire application (markup, styles, and logic in one file).
- `firestore.rules` / `firestore.indexes.json` / `firebase.json` / `.firebaserc` — Firestore configuration and security rules for the `aps-cps-scheduler` Firebase project.
- `.claude/launch.json` — local static-server config for previewing the app.
