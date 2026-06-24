# Cancellation Pattern Analysis — CLAUDE.md

## What this project does

Analyzes a Cordoba lead/enrollment CSV export to surface cancellation patterns across agents, campaigns, credit profiles, states, and enrollment cohorts. Output is an interactive HTML dashboard embedded in the conversation.

---

## Data source

- **File type:** CSV export from Cordoba CRM
- **One row = one lead record** (not one enrolled client — the file includes non-enrolled leads)

---

## Column reference

| Column | What it means |
|---|---|
| `Created` | Timestamp the lead record was created in the system (date + time) |
| `Assigned To` | The closing/enrolling agent — always the agent who enrolled the client |
| `ID` | Internal lead ID |
| `Full Name` | Client name |
| `Home Phone` | Client phone |
| `Enrolled Debt` | Dollar amount enrolled (formatted as `$XX,XXX.00`) |
| `Campaign` | The dialer campaign the call came through — NOT a lead vendor name |
| `Cordoba Enrolled Date` | Date Cordoba officially enrolled the client |
| `Cordoba Dropped Date` | Date Cordoba dropped/cancelled the client |
| `Credit Score` | Credit score pulled from lead data at intake (not verified later) |
| `Total Income` | Client's **monthly** income (formatted as `$X,XXX.00`) |
| `State` | Client's state abbreviation |

---

## Definitions — use these exactly, no exceptions

| Term | Definition |
|---|---|
| **Active Enrolled** | Row has `Cordoba Enrolled Date` AND no `Cordoba Dropped Date` |
| **Cancelled** | Row has both `Cordoba Enrolled Date` AND `Cordoba Dropped Date` |
| **Cancellation Ratio** | `Cancelled ÷ (Active Enrolled + Cancelled)` |
| **Enrolled universe** | Only rows where `Cordoba Enrolled Date` is not null — all analysis is scoped to this subset |

---

## Analysis scope

When the user uploads a CSV and specifies a date range, filter the **enrolled universe** to rows where `Cordoba Enrolled Date` falls within the specified range (inclusive). Then run all analyses below on that filtered set only.

---

## Required analyses — run all of these every time

### 1. Portfolio overview (4 KPI cards)
- Total enrolled (count)
- Cancellation ratio (%)
- Total debt lost to cancellations ($)
- Median days from enrollment to cancellation (days)

### 2. Enrollment cohort trend
Group by `Cordoba Enrolled Date` month. For each month show:
- Total enrolled
- Cancelled count
- Cancel rate %

Display as dual-axis chart: cancel rate % (line, left axis) + enrolled count (bar, right axis).

⚠️ Always include this note: recent cohorts have artificially low cancel rates because clients haven't had time to cancel yet. The Feb–Apr cohorts are the most mature signal.

### 3. Campaign cancellation rates
Group by `Campaign`. Filter to campaigns with ≥ 30 enrollments. Show cancel rate % sorted descending. Color code: ≥35% = red, 25–35% = amber, <25% = green.

### 4. Days-to-cancellation distribution
For cancelled clients only, bucket `Cordoba Dropped Date − Cordoba Enrolled Date` into: 0–3d, 4–7d, 8–14d, 15–30d, 31–60d, 61d+. Show as bar chart. Highlight that 0–14 days is the critical intervention window.

### 5. Credit score × cancel rate
Buckets: <600, 600–650, 650–700, 700+. Show cancel rate per bucket. Flag the counterintuitive finding: higher credit scores cancel MORE (clients with 700+ scores have more financial options and shop around post-enrollment).

### 6. Credit score × monthly income heatmap
Cross-tab: CS buckets (<600, 600–650, 650–700, 700+) × income buckets (<$3k, $3–5k, $5–10k, $10k+/mo). Each cell = cancel rate %. Color gradient from green (low) to red (high).

### 7. Agent cancellation rates
Group by `Assigned To`. Filter to agents with ≥ 10 enrollments. Show cancel rate % sorted ascending (best to worst, bottom to top on horizontal bar chart). Color: <25% = green, 25–35% = amber, >35% = red.

### 8. Key patterns summary
4 insight boxes at the bottom:
- 🔴 Red border: most critical cancellation risk finding
- 🟠 Amber border: campaign-level pattern
- 🟠 Amber border: credit score finding
- 🟢 Green border: top-performing agents to study/replicate

---

## Visualization rules

- Dark ops console aesthetic: dark background (#0f1117), cyan accents (#00d4aa), monospace data labels
- OR: clean white/light dashboard — match the aesthetic already established in the conversation
- Chart.js via CDN for all charts
- All charts rendered inside a single HTML widget via `show_widget`
- Canvas must have `role="img"` and `aria-label` with data summary
- No hardcoded colors that break in dark mode — use Chart.js hex values, not CSS variables, inside canvas
- Custom HTML legends (not Chart.js default)
- Round all displayed numbers: counts = integers, rates = 1 decimal %, dollar values = `$X.XM` or `toLocaleString()`

---

## What NOT to do

- Do not analyze rows where `Cordoba Enrolled Date` is null — these are non-enrolled leads
- Do not treat `Total Income` as annual — it is monthly
- Do not interpret June (or the most recent month) cancel rate as an improvement signal without noting it's immature
- Do not assume `Campaign` = lead vendor — it is the dialer campaign name
- Do not ask clarifying questions about column definitions — they are defined above
- Do not split the output across multiple widgets — one single dashboard widget

---

## Files in this project

| File | Purpose |
|---|---|
| `CLAUDE.md` | This file — project memory and analysis spec |
| `cancellation_analysis_prompt.md` | The prompt to paste when uploading a new CSV |

---

## Notes from initial analysis (June 2026 baseline)

- Portfolio: 4,466 enrolled, 28.2% cancel rate, $45.2M debt lost
- AIM-family campaigns: 33–47% cancel rate (benchmark: TransferK at 21.6%)
- 700+ credit score clients cancel at 36–43% — highest-risk profile despite appearing most qualified
- 49% of cancellations happen within 14 days of enrollment
- Best agents: Omar Tayara (6.1%), Adam Elqaza (14.9%), Eileen Beeson (19%)
- Worst agents: Joseph Nassif (53.3%), Anthony Dimora (50%)
