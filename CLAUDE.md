# ADP Cancellation Dashboard — CLAUDE.md

## Project Overview
Single-file HTML dashboard (`index.html`) for analysing ADP/Cordoba client enrollment and cancellation data from a CRM CSV export. No build step, no server — open `index.html` directly in a browser.

## Tech Stack
- **PapaParse 5.4.1** — CSV parsing (CDN)
- **Chart.js 4.4.1** — all charts (CDN)
- Pure vanilla HTML/CSS/JavaScript — no framework, no npm

## Expected CSV Columns
These exact column names are auto-detected (with BOM/invisible-char stripping):

| Field | CSV Column Name |
|---|---|
| Cordoba Enrolled Date | `Cordoba Enrolled Date` |
| Cordoba Dropped Date | `Cordoba Dropped Date` |
| Agent Name | `Assigned To` |
| State | `State` |
| Created Date | `Created` |
| Enrolled Debt | `Enrolled Debt` |

Detection uses exact-match first, then regex pattern fallback. Only rows with a non-empty `Cordoba Enrolled Date` are analysed.

## Metric Definitions
- **Active Enrolled** — has a Cordoba Enrolled Date, no Cordoba Dropped Date
- **Cancelled** — has both Enrolled Date and Dropped Date
- **Cancellation Ratio** — Cancelled ÷ (Active Enrolled + Cancelled)

## Key Features
- Drag-and-drop CSV upload
- Date cohort filter (by enrolled date), state filter, agent filter
- KPI summary cards (Total, Active, Cancelled, Cancel Rate)
- Auto-generated Key Insights (worst/best state, worst/best agent, cancellation trend, median days to cancel, debt comparison)
- Charts: Monthly enrolments vs cancellations, Cancel ratio trend, Cancellations by state, Cancel rate by state, Debt vs cancel rate, Days-to-cancel distribution, Day-of-week patterns
- Sortable agent and state breakdown tables (click any column header)

## Development Notes
- All logic lives in a single `<script>` block at the bottom of `index.html`
- `colMap` holds the detected column mapping: `{ enrolledDate, droppedDate, agent, state, createdDate, debtAmount }`
- `rawRows` holds all parsed CSV rows; filters re-run `renderDashboard(filteredRows)`
- Charts are stored in the `charts` object and destroyed/recreated on each filter change
- `chartEmpty(canvasId, msg)` shows an overlay message when a required column is missing
- Sort state per table is tracked in `sortState[tableId] = { col, dir }`
- Branch for development: `claude/kind-gates-ywny4z`
