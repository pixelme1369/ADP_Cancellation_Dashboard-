# CRM CSV Export — Reference

This document describes the CSV file used by the ADP Cancellation Dashboard (`ADP_Cancellation_Dashboard.html`). It's a CRM export of ADP/Cordoba client enrollment and cancellation data.

## Required Columns

These columns are auto-detected by exact name match (after stripping BOM/invisible characters):

| Field | CSV Column Name | Notes |
|---|---|---|
| Cordoba Enrolled Date | `Cordoba Enrolled Date` | Only rows with a non-empty value here are analysed at all |
| Cordoba Dropped Date | `Cordoba Dropped Date` | Empty = still active; filled = cancelled |
| Agent Name | `Assigned To` | The agent assigned to the client |
| State | `State` | Client's state |
| Created Date | `Created` | Record creation date |
| Enrolled Debt | `Enrolled Debt` | Dollar amount, may include `$` and `,` |
| Zip Code | `Zip` | Client's zip code (5-digit or ZIP+4, e.g. `75001` or `75001-1234`) |

## Optional Columns (detected by pattern, not exact name)

These are detected via regex against the header row, so naming can vary slightly:

| Field | Matches headers containing |
|---|---|
| Campaign / Source | `campaign`, `source`, `lead source`, `channel` |
| Credit Score | `credit score`, `fico` |
| Monthly Income | `monthly income`, `income` |

If a column isn't found, the corresponding chart/section on the dashboard shows an empty-state message rather than failing.

## Date Format

Dates in the CRM export use `M/D/YY` or `M/D/YYYY`, optionally with a time component (e.g. `6/19/26`, `6/19/2026 13:25`). Two-digit years are treated as `20YY`. Standard JavaScript date parsing (`new Date(string)`) is **not** reliable for this format and is not used directly.

## Row Filtering

Only rows with a non-empty `Cordoba Enrolled Date` are included in analysis — rows without an enrolled date are dropped entirely on load.

## Metric Definitions

- **Active Enrolled** — has a Cordoba Enrolled Date, no Cordoba Dropped Date
- **Cancelled** — has both an Enrolled Date and a Dropped Date
- **Cancellation Ratio** — Cancelled ÷ (Active Enrolled + Cancelled)

## Zip Code Handling

Zip codes are normalised to their first 5 digits, so ZIP+4 values (e.g. `75001-1234`) are grouped together with their base 5-digit zip (`75001`) in all charts and tables.
