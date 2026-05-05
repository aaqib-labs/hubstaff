# Hubstaff Monthly Dashboards — Claude Briefing

## What This Project Does
Converts raw Hubstaff time-tracking CSV exports into standalone, interactive HTML performance dashboards for WebLife departments. Each dashboard is a self-contained HTML file with embedded React components and JSON data. Dashboards are committed to GitHub and shared with managers.

**GitHub repo:** https://github.com/aaqib-labs/hubstaff
**Owner:** Aaqib Hafeel (aaqib@weblifestores.com) — WebLife

---

## Folder Structure

```
/
├── index.html                  → Master hub page linking to all department hubs
├── CLAUDE.md                   → This file
├── .gitignore
├── data/                       → Raw Hubstaff CSVs (local only, gitignored)
│   └── YYYY-MM-master.csv
├── hubs/                       → Department hub pages (one per department)
│   ├── task-agency.html
│   ├── allintalent.html
│   ├── digital-product.html
│   ├── media.html
│   ├── marketing-media.html
│   └── venia-products.html
├── template/
│   ├── dashboard-template.html → Single-tab dashboard (most departments)
│   ├── template-2tab.html      → Two-tab dashboard
│   └── template-3tab.html      → Three-tab dashboard (e.g. Digital Product, Venia)
└── dashboards/
    ├── task-agency/            → Task Agency monthly dashboards
    ├── allintalent/            → allintalent monthly dashboards
    ├── digital-product/        → Digital Product monthly dashboards
    ├── media/                  → Media monthly dashboards
    ├── marketing-media/        → Marketing + Media monthly dashboards
    └── venia-products/         → Venia Products monthly dashboards
```

---

## Departments & File Naming

| Department | File Slug | Template |
|---|---|---|
| Task Agency | `task-agency` | `template-2tab.html` (2 tabs: Overview / Heatmap) |
| allintalent | `allintalent` | `template-2tab.html` (2 tabs: Overview / Heatmap) |
| Digital Product | `digital-product` | `template-3tab.html` (3 tabs: Overview / Heatmap / Team Analytics — has sub-teams) |
| Media | `media` | `template-2tab.html` (2 tabs: Overview / Heatmap) |
| Marketing + Media | `marketing-media` | `template-2tab.html` (2 tabs: Overview / Heatmap) |
| Venia Products | `venia-products` | `template-3tab.html` (3 tabs: Overview / Heatmap / Team Analytics — has sub-teams) |

**File naming convention:** `{slug}-YYYY-MM.html` — e.g. `task-agency-2026-03.html`

---

## CSV Data Format

Raw file location: `data/YYYY-MM-master.csv`

**Columns in the CSV:**
- `Team(s)` — department/team name (used to filter rows per department)
- `Member` — employee name
- `Activity %` — Hubstaff activity percentage
- `Total Worked Hours` — total hours tracked
- `Break Time` — break hours
- `Break % of Total` — break as % of worked hours
- `Total Manual Hours` — manually entered hours
- `Manual % of Total` — manual as % of worked hours
- `Low Activity Hours (≤20%)` — hours where activity was ≤20%
- `Low Activity % (≤20%)` — those hours as % of total
- `Low Activity Hours (≤30%)` — hours where activity was ≤30%
- `Low Activity % (≤30%)` — those hours as % of total
- `SLA Violation Legend` — calculated SLA flags, format: `MetricCode🔴/⚠️/🟠`, comma-separated (e.g. `A🔴, H🔴, 20⚠️`). Blank if fully compliant.
- `Red Flag Count` — number of 🔴 flags
- `Yellow Flag Count` — number of ⚠️ flags only (orange `H🟠` is NOT counted here — it counts with red)
- `Total Flags` — Red Flag Count + Yellow Flag Count

> **Note:** `Total Manual Hours` and `Manual % of Total` are included in the master CSV but are **never shown in dashboards**. They are collected for record-keeping only.

---

## SLA Thresholds (Hardcoded in Every Dashboard)

| Metric | Red | Yellow | Orange |
|---|---|---|---|
| Activity % | < 35% | < 45% | — |
| Total Hours | < 160 hrs | — | ≥ 200 hrs |
| Break % | ≥ 12% | > 10% | — |
| Manual % | ≥ 10% | ≥ 5% | — |
| Low Activity ≤20% | ≥ 15% | ≥ 7.5% | — |
| Low Activity ≤30% | ≥ 20% | ≥ 10% | — |

---

## Building the Master CSV (`data/YYYY-MM-master.csv`)

Before building any dashboards, you need to create the master CSV by merging 5 Hubstaff exports.

### Step 1 — Export these 5 reports from Hubstaff

| # | Hubstaff Report Type | Save as (in `~/Downloads/`) |
|---|---|---|
| 1 | Hours + Break + Activity % (grouped by member, full month) | `[Month] '[YY] Hrs, Activity, Break time.csv` |
| 2 | Manual Hours (grouped by member, full month) | `[Month] '[YY] Manual hrs.csv` |
| 3 | Low Activity ≤20% hours (grouped by member, full month) | `[Month] '[YY] Act <20%.csv` |
| 4 | Low Activity ≤30% hours (grouped by member, full month) | `[Month] '[YY] Act <30%.csv` |
| 5 | Team Overview / time by member + team (full month) | `[Month] '[YY] team overview.csv` |

**Example for March 2026:** `March '26 Hrs, Activity, Break time.csv`, `March '26 Manual hrs.csv`, etc.

### Step 2 — Column mapping

| Master Column | Source File | Source Column |
|---|---|---|
| `Member` | File 1 | `Grouped by Member` |
| `Total Worked Hours` | File 1 | `Time` |
| `Break Time` | File 1 | `Total Break` |
| `Activity %` | File 1 | `Average Activity` |
| `Total Manual Hours` | File 2 | `Total Manual Hours` |
| `Low Activity Hours (≤20%)` | File 3 | `Time` |
| `Low Activity Hours (≤30%)` | File 4 | `Time` |
| `Team(s)` | File 5 | `Teams` (unique teams per member, comma-joined) |

All percentage columns (`Break % of Total`, `Manual % of Total`, `Low Activity % (≤20%)`, `Low Activity % (≤30%)`) are **calculated** as `(numerator ÷ Total Worked Hours) × 100`.

Members not present in Files 3 or 4 get `0.0` for those low-activity columns.

### Step 3 — SLA flag logic (applied during build)

| Flag | Code | Condition |
|---|---|---|
| Activity red | `A🔴` | Activity % < 35% |
| Activity yellow | `A⚠️` | Activity % < 45% (and ≥ 35%) |
| Hours red | `H🔴` | Total Worked Hours < 160 |
| Hours orange | `H🟠` | Total Worked Hours ≥ 200 — **counted together with red in the "Red/Orange Flags" total** |
| Break red | `B🔴` | Break % ≥ 12% |
| Break yellow | `B⚠️` | Break % > 10% (and < 12%) |
| Manual red | `M🔴` | Manual % ≥ 10% |
| Manual yellow | `M⚠️` | Manual % ≥ 5% (and < 10%) |
| Low Act ≤20% red | `20🔴` | Low Activity % (≤20%) ≥ 15% |
| Low Act ≤20% yellow | `20⚠️` | Low Activity % (≤20%) ≥ 7.5% (and < 15%) |
| Low Act ≤30% red | `30🔴` | Low Activity % (≤30%) ≥ 20% |
| Low Act ≤30% yellow | `30⚠️` | Low Activity % (≤30%) ≥ 10% (and < 20%) |

`Red Flag Count` = count of 🔴 flags **plus** 🟠 orange. `Yellow Flag Count` = count of ⚠️ flags only. The dashboard label reads "Red/Orange Flags" to reflect this.

**SLA Violation Legend format:** `MetricCode + emoji`, comma-separated, reds first then yellows then orange. Leave blank if fully compliant.
```
A🔴, H🔴, 20🔴        ← multiple reds
H🟠, B⚠️              ← orange + yellow
30🔴, A⚠️             ← mixed
                       ← blank = compliant
```

### Step 4 — Run the Python build script

When the user says they have exported the 5 Hubstaff reports for a new month, run this Python script (adjusting `month_prefix` and `out_path`):

```python
import pandas as pd
import numpy as np

month_prefix = "March '26"   # <-- change each month
base = "/Users/Weblife/Downloads/"
out_path = "data/2026-03-master.csv"  # <-- change each month

def read_member_file(path):
    df = pd.read_csv(path, index_col=False)
    if df.index.dtype == object:
        df = df.reset_index().rename(columns={'index': 'Member'})
    else:
        df = df.rename(columns={df.columns[0]: 'Member'})
    return df

hrs_df    = read_member_file(f"{base}{month_prefix} Hrs, Activity, Break time.csv")
manual_df = read_member_file(f"{base}{month_prefix} Manual hrs.csv")
act20_df  = read_member_file(f"{base}{month_prefix} Act <20%.csv")
act30_df  = read_member_file(f"{base}{month_prefix} Act <30%.csv")
team_df   = pd.read_csv(f"{base}{month_prefix} team overview.csv")

hrs_df = hrs_df.rename(columns={
    'Time': 'Total Worked Hours', 'Total Break': 'Break Time', 'Average Activity': 'Activity %'
})[['Member', 'Total Worked Hours', 'Break Time', 'Activity %']]

manual_df = manual_df[['Member', 'Total Manual Hours']]
act20_df  = act20_df.rename(columns={'Time': 'Low Activity Hours (≤20%)'})[['Member', 'Low Activity Hours (≤20%)']]
act30_df  = act30_df.rename(columns={'Time': 'Low Activity Hours (≤30%)'})[['Member', 'Low Activity Hours (≤30%)']]

teams_per_member = (
    team_df.groupby('Member')['Teams']
    .apply(lambda x: ', '.join(sorted(set(x.dropna().astype(str)))))
    .reset_index().rename(columns={'Teams': 'Team(s)'})
)

df = hrs_df.copy()
for right in [manual_df, act20_df, act30_df, teams_per_member]:
    df = df.merge(right, on='Member', how='left')

for col in ['Total Manual Hours', 'Low Activity Hours (≤20%)', 'Low Activity Hours (≤30%)']:
    df[col] = df[col].fillna(0.0)
for col in ['Total Worked Hours', 'Break Time', 'Total Manual Hours',
            'Low Activity Hours (≤20%)', 'Low Activity Hours (≤30%)']:
    df[col] = df[col].round(1)

df['Activity %'] = df['Activity %'].astype(str).str.replace('%', '').str.strip().astype(float)

def pct(num, den):
    return np.where(den > 0, (num / den * 100).round(1), 0.0)

df['Break % of Total']      = pct(df['Break Time'],                df['Total Worked Hours'])
df['Manual % of Total']     = pct(df['Total Manual Hours'],         df['Total Worked Hours'])
df['Low Activity % (≤20%)'] = pct(df['Low Activity Hours (≤20%)'],  df['Total Worked Hours'])
df['Low Activity % (≤30%)'] = pct(df['Low Activity Hours (≤30%)'],  df['Total Worked Hours'])

def sla_flags(row):
    act  = float(row['Activity %'])
    hrs  = row['Total Worked Hours']
    brk  = row['Break % of Total']
    man  = row['Manual % of Total']
    la20 = row['Low Activity % (≤20%)']
    la30 = row['Low Activity % (≤30%)']
    reds, yellows, oranges = [], [], []

    if act < 35:      reds.append('A🔴')
    elif act < 45:    yellows.append('A⚠️')
    if hrs < 160:     reds.append('H🔴')
    elif hrs >= 200:  oranges.append('H🟠')  # counted with red in Red/Orange Flags total
    if brk >= 12:     reds.append('B🔴')
    elif brk >= 10:   yellows.append('B⚠️')
    if man >= 10:     reds.append('M🔴')
    elif man >= 5:    yellows.append('M⚠️')
    if la20 >= 15:    reds.append('20🔴')
    elif la20 >= 7.5: yellows.append('20⚠️')
    if la30 >= 20:    reds.append('30🔴')
    elif la30 >= 10:  yellows.append('30⚠️')

    legend = ', '.join(reds + yellows + oranges) if (reds or yellows or oranges) else ''
    return pd.Series([legend, len(reds), len(yellows), len(reds) + len(yellows)])

df[['SLA Violation Legend', 'Red Flag Count', 'Yellow Flag Count', 'Total Flags']] = df.apply(sla_flags, axis=1)

df['Activity %'] = df['Activity %'].apply(lambda x: f"{int(x)}%")
for col in ['Break % of Total', 'Manual % of Total', 'Low Activity % (≤20%)', 'Low Activity % (≤30%)']:
    df[col] = df[col].apply(lambda x: f"{x}%")

cols = [
    'Team(s)', 'Member', 'Activity %',
    'Total Worked Hours', 'Break Time', 'Break % of Total',
    'Total Manual Hours', 'Manual % of Total',
    'Low Activity Hours (≤20%)', 'Low Activity % (≤20%)',
    'Low Activity Hours (≤30%)', 'Low Activity % (≤30%)',
    'SLA Violation Legend', 'Red Flag Count', 'Yellow Flag Count', 'Total Flags'
]
df = df[cols].sort_values('Member').reset_index(drop=True)  # alphabetical by member name
df.to_csv(out_path, index=False)
print(f"Saved {len(df)} members → {out_path}")
```

**Formatting rules enforced by the script:**
- All hour values rounded to 1 decimal place
- All percentage columns formatted with `%` symbol (e.g. `9.3%`)
- Table sorted **alphabetically by Member name**
- All members from the dataset included, even those with no team data
- Manual hours columns present in the CSV but **excluded from all dashboard data blocks**

---

## Monthly Workflow (Adding a New Month's Dashboards)

1. **Build the master CSV** — Export the 5 Hubstaff reports and run the Python script described in the "Building the Master CSV" section above. Output goes to `data/YYYY-MM-master.csv`.
2. **Extract department data** — Use Python to filter rows by `Team(s)` column for each department
3. **Pick the right template** — See template column in the department table above
4. **Populate the dashboard** — Copy the template, embed the JSON data in the `DATA BLOCK` section, fill in month/year/member counts
5. **Replace all template placeholders** — `{{DEPT}}`, `{{MONTH}}`, `{{YEAR}}`, and **`{{DEPT_HUB_LINK}}`** (e.g. `../../hubs/task-agency.html`). The `{{DEPT_HUB_LINK}}` powers the back nav bar — if left unfilled it will be a broken link.
6. **Save to `dashboards/{slug}/`** — e.g. `dashboards/task-agency/task-agency-2026-03.html`
7. **Update the department hub page** (`hubs/{slug}.html`) — promote the new month to "Latest Report", move the previous latest down to "Previous Reports"; update the `index.html` card-meta months line (e.g. `Jan · Feb · Mar · Apr 2026`)
8. **Commit and push** — Use descriptive commit messages; credit Claude as co-author

The data block in each dashboard is marked with:
```
// THIS BLOCK IS AUTO-POPULATED BY CLAUDE EACH MONTH — DO NOT EDIT MANUALLY
```

---

## Tech Stack

- **React 18.2.0** (via CDN, Babel transpiles JSX in-browser)
- **Font:** DM Sans + DM Mono (Google Fonts)
- **No build step** — files open directly in a browser or are served as static files
- **No backend** — everything is embedded in the HTML file

---

## Template Guide

| Template file | Tabs | When to use |
|---|---|---|
| `dashboard-template.html` | 3 (Overview / Heatmap / Comparisons) | Departments with no sub-teams |
| `template-2tab.html` | 2 (Overview / Heatmap) | Departments that only need two views |
| `template-3tab.html` | 3 (Overview / Heatmap / Team Analytics) | Departments with sub-teams |

---

## Common Tasks

- **New month:** Follow the Monthly Workflow above for each department
- **New department:** Create a new dashboard from the appropriate template, add to index.html, add to the department table in this file
- **Fix a dashboard:** Read the existing `.html` file, identify the issue, edit the data block or component logic
- **Update SLA thresholds:** Change in each dashboard file AND update the table in this CLAUDE.md

---

## Notes for Claude

- Always read the existing dashboard for the same department from the previous month before building a new one — the sub-team structures and member lists carry over (with updates)
- **Never include `Total Manual Hours` or `Manual % of Total` in dashboard data blocks** — these columns exist in the master CSV for record-keeping only
- The `settings.local.json` has Python and git permissions already configured
- When using Python to extract CSV data, filter by the `Team(s)` column
- Always update `index.html` after adding new dashboards
- Commit with descriptive messages and add co-author line: `Co-Authored-By: Claude Sonnet 4.6 (1M context) <noreply@anthropic.com>`
