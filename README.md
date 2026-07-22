# LBAC Dashboards

Public landing page + dashboards for Council 032 Long Beach, CA.

**Live site:** https://kevinfolkerts.github.io/LBACDashboards/

## Daily workflow (the short version)

1. Download data exports (MyScouting, BlackPug, BlackBaud, CRS) into
   **`C:\Users\kdfol\Documents\LBAC Dashboard Inbox`** — all of them, any mix.
2. Double-click **`LBAC_Update.bat`** (here) or **`UPDATE DASHBOARDS.bat`**
   (inside the inbox). It sorts every file to its data folder, refreshes the
   affected dashboards, and offers to publish the site.

That's it. Details below.

## Repository / folder contents

| File | Purpose |
|---|---|
| `index.html` | Landing page — select a dashboard |
| `LBAC_Update.bat` | **One-click pipeline**: sort inbox → refresh dashboards → publish |
| `run_pipeline.py` | The pipeline behind LBAC_Update.bat |
| `sort_inbox.py` + `routing_rules.json` | Inbox sorter + editable filename→folder rules |
| `refresh_<name>.py` | Per-dashboard data refresh (membership, socal, seabase, tahquitz, webelos ×2) |
| `lbac_paths.json` | Where each dashboard HTML + data folder lives |
| `publish.py` / `publish.bat` | Push dashboards + landing page to GitHub Pages |
| `*.lnk` / `*.target` | Pointers to the dashboard HTML files publish.py collects |
| `DASHBOARD_TEMPLATE_SPEC.md` | Conventions + checklist for building new dashboards |
| `_backups\` | Timestamped originals from tooling edits |
| `.github/workflows/build.yml` | Optional CI fallback |

## Sorting rules

**my.scouting.org downloads need no renaming.** Drop `data.csv`,
`data2.csv`, `dataadult.csv`, etc. straight into the inbox — the sorter
identifies each report by its contents (snapshot vs monthly-trend columns,
single-council vs 9-council) and renames it to your convention
(`Jul26.csv`, `Jul26B.csv`, `Jul26 Adults.csv`, `Dropped 2026.csv`, …).
The month/year comes from the file's download date. New Member / Dropped /
Lapsed / Crossover share an identical format, so those are recognized by
matching against your existing files for the same year; if the sorter can't
tell (e.g. the first download of a new year), it asks you right in the
console window.

`routing_rules.json` maps filename patterns to destination folders (first
match wins). Same-named files at the destination are archived to
`_Archive\<name>__replaced_<timestamp>.csv` before the new file takes over.
Unrecognized files land in the inbox's `_Unsorted\` folder. Every action is
logged to the inbox's `sort_log.txt`. To route files for a future dashboard,
add a rule to the JSON — no code changes.

## Refreshing dashboards

Each `refresh_<name>.py` re-parses the CSVs in that dashboard's data folder
and regenerates the data embedded in the dashboard HTML (between
`/*LBAC_DATA_START*/` markers). Run standalone any time:

```
py -3 refresh_membership.py            (uses lbac_paths.json defaults)
py -3 refresh_membership.py --check    (validate only, write nothing)
```

**Giving Dashboard exception:** donor files are sorted automatically, but the
Giving Dashboard itself is refreshed in a Cowork chat with Claude.

## Publishing updates

`LBAC_Update.bat` offers to publish at the end, or run `publish.bat` any
time. `publish.py` collects dashboards from the `.lnk` shortcuts and
`.target` files here, stamps each with its data date (from the
`lbac-data-date` meta tag), fills the landing-page cards, and pushes to
GitHub Pages. The site updates within ~1 minute.

## One-time setup (new machine)

1. **Clone the repo**
   ```
   cd C:\Users\kdfol\Documents
   git clone https://github.com/KevinFolkerts/LBACDashboards.git
   ```
2. **Enable GitHub Pages** — Repo Settings → Pages → Deploy from a branch →
   `main` / root. Site: https://kevinfolkerts.github.io/LBACDashboards/
3. **Git auth** — first push may prompt for credentials; use a Personal
   Access Token if 2FA is on (Settings → Developer settings → Tokens →
   "repo" scope).

## Fully automatic?

Schedule the pipeline via Windows Task Scheduler:

1. Create Basic Task → "LBAC Dashboard Update"
2. Trigger: Daily at 7 AM
3. Action: Start a program
   - Program: `py`
   - Arguments: `-3 run_pipeline.py --publish`
   - Start in: `C:\Users\kdfol\Documents\MyScouting Reports\Membership Totals\LBAC_Publish`

Anything dropped in the inbox gets sorted, refreshed, and published every
morning.
