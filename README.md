# ADP Cancellation Dashboard

A single-file, zero-dependency dashboard for analysing client enrollment and cancellation data from a CRM CSV export.

---

## Getting Started

1. Download or clone this repository
2. Open `index.html` in any modern browser
3. Drag & drop your CRM CSV export onto the upload zone (or click to browse)

No server, no install, no build step required.

---

## What It Analyses

The dashboard filters down to rows that have a **Cordoba Enrolled Date** and computes:

| Metric | Definition |
|---|---|
| Active Enrolled | Has an enrolled date, no dropped date |
| Cancelled | Has both an enrolled date and a dropped date |
| Cancellation Ratio | Cancelled ÷ (Active + Cancelled) |

---

## Expected CSV Columns

The parser auto-detects these column names:

| Field | Expected Column |
|---|---|
| Enrollment date | `Cordoba Enrolled Date` |
| Cancellation date | `Cordoba Dropped Date` |
| Agent / rep | `Assigned To` |
| Client state | `State` |
| Lead created | `Created` |
| Debt amount | `Enrolled Debt` |

If a column isn't found automatically, charts that depend on it will show a "not detected" message overlay.

---

## Features

### Filters
- **Date range** — slice by Cordoba Enrolled Date
- **State** — filter to a single state
- **Agent** — filter to a single agent

### KPI Cards
- Total rows analysed, Active Enrolled, Cancelled, Cancellation Ratio

### Key Insights (auto-generated)
- Worst and best state by cancel rate
- Worst and best agent by cancel rate
- Peak day of week for enrolments and cancellations
- Median days to cancellation and % leaving within 30 days
- Average enrolled debt: cancelled vs active clients
- Cancel rate trend (improving or worsening over time)

### Charts
| Chart | What it shows |
|---|---|
| Monthly Enrolments vs Cancellations | Volume by month |
| Cancellation Ratio by Enrolment Month | Rate trend over time |
| Cancellations by State (Top 15) | Which states generate the most cancels |
| Cancel Rate by State (Top 15) | Which states have the worst retention |
| Cancel Rate by Enrolled Debt Range | Does debt size predict cancellation? |
| Days to Cancellation Distribution | How quickly do clients cancel? |
| Enrolments by Day of Week | When do clients enrol? |
| Cancellations by Day of Week | When do clients drop? |

### Tables
- **Agent Rankings** — sortable by enrolled, active, cancelled, cancel rate
- **State Breakdown** — sortable by all columns; click any header to sort

---

## Tech
- [PapaParse](https://www.papaparse.com/) — CSV parsing
- [Chart.js](https://www.chartjs.org/) — charts
- Vanilla HTML/CSS/JavaScript — no framework or build tool
