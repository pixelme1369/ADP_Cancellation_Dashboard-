# ADP Cancellation Dashboard — CLAUDE.md

## Project Overview
Single-file HTML dashboard (`ADP_Cancellation_Dashboard.html`) for analysing ADP/Cordoba client enrollment and cancellation data from a CRM CSV export. No build step, no server — open the file directly in a browser.

## Tech Stack
- **PapaParse 5.4.1** — CSV parsing (CDN)
- **Chart.js 4.4.1** — all charts (CDN)
- Pure vanilla HTML/CSS/JavaScript — no framework, no npm

## Expected CSV Columns
These exact column names are auto-detected (with BOM/invisible-char stripping via `cleanH()`):

| Field | CSV Column Name |
|---|---|
| Cordoba Enrolled Date | `Cordoba Enrolled Date` |
| Cordoba Dropped Date | `Cordoba Dropped Date` |
| Agent Name | `Assigned To` |
| State | `State` |
| Created Date | `Created` |
| Enrolled Debt | `Enrolled Debt` |
| Zip Code | `Zip` |

Additional columns detected by regex pattern fallback: `campaign`, `creditScore`, `monthlyIncome`.

Detection uses exact-match first (after `cleanH()` stripping), then regex fallback. Only rows with a non-empty `Cordoba Enrolled Date` are analysed.

## CSV Date Format
The CRM export uses `M/D/YY` or `M/D/YYYY` format (e.g. `6/19/26`, `6/19/2026 13:25`). The `parseDate()` function handles this explicitly — 2-digit years are treated as 2000+. Do **not** rely on `new Date(string)` directly for these values.

## Metric Definitions
- **Active Enrolled** — has a Cordoba Enrolled Date, no Cordoba Dropped Date
- **Cancelled** — has both Enrolled Date and Dropped Date
- **Cancellation Ratio** — Cancelled ÷ (Active Enrolled + Cancelled)

## Layout — FINDING N Sections
The dashboard is structured as numbered findings with dynamic auto-generated titles:

| Section | Content |
|---|---|
| PORTFOLIO OVERVIEW | 4-card KPI grid (Total, Active, Cancelled, Rate) with sub-labels |
| FINDING 1 | Dual-axis cohort chart (enrollments + cancellations bar, cancel rate % line) |
| FINDING 2 | Campaign/source inline horizontal bars, color-coded by cancel rate |
| FINDING 3 | Days-to-cancellation bar chart + two callout boxes (early exits, median) |
| FINDING 4A | Cancel rate by credit score band |
| FINDING 4B | State × debt range heatmap (top 10 states) |
| FINDING 5 | Agent horizontal bar chart, auto-sized height by agent count |
| FINDING 6 | Zip code horizontal bar chart (top 20 by cancel rate, min 3 enrollments/zip, labelled with state), auto-sized height |
| KEY PATTERNS SUMMARY | 4 narrative blocks with colored left borders |
| Tables | Sortable agent rankings + state breakdown side by side, plus a full-width zip code breakdown table (zip, state, enrolled, active, cancelled, rate) |

## Development Notes
- All logic lives in a single `<script>` block at the bottom of `ADP_Cancellation_Dashboard.html`
- `colMap` holds detected column mapping: `{ enrolledDate, droppedDate, agent, state, createdDate, debtAmount, zipCode, campaign, creditScore, monthlyIncome }`
- `rawRows` holds all parsed CSV rows; filters re-run `renderDashboard(filteredRows)`
- `parseDate(s)` — handles `M/D/YY`, `M/D/YYYY`, and ISO formats; returns a `Date` or `null`
- `zip5(z)` — normalises a zip code to its first 5 digits (collapses `ZIP+4` values like `75001-1234` into `75001`)
- `cleanH(h)` — strips BOM/zero-width chars (U+200B, U+200C, U+200D, U+FEFF, U+00A0) but NOT regular spaces
- Charts are stored in `charts` object and destroyed/recreated on each render via `destroyChart(id)`
- `chartEmpty(canvasId, msg)` — shows overlay message on canvas when column is missing or data is empty
- Sort state per table tracked in `sortState[tableId] = { col, dir }`; per-table column keys live in `sortTbl()`'s `keysMap`/`rowFns` (`tblAgent`, `tblState`, `tblZip`)
- `applyFilters()` compares dates as timestamps (not strings) to handle the M/D/YY format correctly
- `initFilters()` clears date inputs on each CSV load to prevent stale browser-remembered values
- Branch for development: `claude/cancellation-zipcode-state-ogarsb`
