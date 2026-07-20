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

## Layout — Command Center Modules
The dashboard is a "Cancellation Command Center": a sticky command bar (brand, live-status pill with record count + date range, Export CSV, New file) over a sticky control deck (filters + section-jump nav chips), then numbered analytical modules with dynamic auto-generated titles. Dark theme is deliberate (an ops-center surface, not an auto light/dark flip).

| Module | Content |
|---|---|
| 01 PORTFOLIO VITALS | 6-card KPI strip (Total, Active, Cancelled, Cancel Rate, Debt at Risk, Median Tenure) with status-colored left borders and sub-labels |
| 02 COHORT TREND | Two **single-axis** charts (no dual-axis): grouped enroll/cancel volume bars + a separate cancel-rate % line |
| 03 TIMING & SURVIVAL | Days-to-cancellation histogram + two callouts (early exits, median), and a cumulative-cancellation % curve |
| 04 SEGMENT RISK | Cancel rate by credit band + by enrolled-debt band (ordinal blue ramps), state × debt heatmap (top 10, green→amber→red), and sortable state table |
| 05 GEOGRAPHY | Zip-code horizontal bar chart (top 20 by cancel rate, min 3 enrollments/zip, labelled with state), auto-sized height, + full-width sortable zip breakdown table (zip, state, enrolled, active, cancelled, rate) |
| 06 SOURCE & CAMPAIGN | Campaign/source inline horizontal bars, color-coded by cancel rate |
| 07 TEAM PERFORMANCE | Agent horizontal bar chart (auto-sized height) + sortable agent rankings table |
| 08 INTELLIGENCE BRIEFING | 4 flagged narrative cards (state / timing / team / portfolio) with colored left borders |

### Data-viz rules honored (dataviz skill)
- **No dual-axis charts.** Cohort volume and cohort rate are two separate one-axis charts.
- Meaning-colors ship with labels/legends, never color-alone: `--blue` enrollments, `--red` cancelled/critical, `--amber` cancel-rate/caution, `--green` active/retained. Palette CVD-validated against the dark surface.
- Ordinal segment ramps use a single-hue blue ramp; the heatmap is a green→amber→red sequential scale with a legend.

## Development Notes
- All logic lives in a single `<script>` block at the bottom of `ADP_Cancellation_Dashboard.html`
- `colMap` holds detected column mapping: `{ enrolledDate, droppedDate, agent, state, createdDate, debtAmount, zipCode, campaign, creditScore, monthlyIncome }`
- `rawRows` holds all parsed CSV rows; `filteredRows` holds the current slice; filters re-run `renderDashboard(rows)`
- `parseDate(s)` — handles `M/D/YY`, `M/D/YYYY`, and ISO formats; returns a `Date` or `null`
- `zip5(z)` — normalises a zip code to its first 5 digits (collapses `ZIP+4` values like `75001-1234` into `75001`)
- `isCancelled(r)` — single source of truth for "has a non-empty Cordoba Dropped Date"
- `cleanH(h)` — strips BOM/zero-width chars (U+200B, U+200C, U+200D, U+FEFF, U+00A0) but NOT regular spaces
- Chart.js is themed once via `Chart.defaults`; per-chart axis defaults live in the `AX` object and colors in `C`
- Cohort series is computed once by `cohortSeries()` and shared by `renderVolumeChart()` and `renderRateChart()`
- Charts are stored in `charts` object and destroyed/recreated on each render via `destroyChart(id)`. Canvas IDs: `chartVolume, chartRate, chartDays, chartCumulative, chartCredit, chartDebt, chartAgent, chartZip`
- `chartEmpty(canvasId, msg)` — shows overlay message on canvas when column is missing or data is empty
- Sort state per table tracked in `sortState[tableId] = { col, dir }`; per-table column keys live in `sortTbl()`'s `keysMap`/`rowFns` (`tblAgent`, `tblState`, `tblZip`)
- `applyFilters()` compares dates as timestamps (not strings); adds a Status filter (all / active / cancelled)
- `initFilters()` clears date inputs on each CSV load (plus `autocomplete="off"` and a next-tick re-clear) to prevent stale browser-remembered values
- `exportCSV()` downloads the current `filteredRows` slice; "New file" resets to the upload screen
- Filters live in the sticky `.deck`; the `.navchip` anchors scroll-jump to each module (`#sec-*`)
- Branch for development: `claude/cancellation-command-center-8iuwtt`
