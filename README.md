# Project 2037 — Retirement Glide Path Dashboard

Two files make up the dashboard:

- `index.html` — the dashboard itself (layout, charts, logic)
- `data.json` — all the numbers (portfolio holdings, plan milestones, scenarios). Edit this file when you want to refresh the data; never need to touch `index.html` again after this initial setup.

## 1. Deploy to GitHub Pages (one-time, ~10 minutes)

1. Go to [github.com/new](https://github.com/new) and create a repository, e.g. `project-2037-dashboard`. Keep it **Public** (Pages on a free personal account requires a public repo) or **Private** with GitHub Pages available if you're on a paid plan.
2. On the new repo's page, click **"uploading an existing file"** (or **Add file → Upload files**).
3. Drag in `index.html` and `data.json` from this folder. Commit directly to `main`.
4. Go to **Settings → Pages** (left sidebar).
5. Under **Build and deployment → Source**, choose **Deploy from a branch**. Under **Branch**, pick `main` and folder `/ (root)`. Click **Save**.
6. Wait ~1 minute, then refresh the Pages settings page — it will show a URL like:
   `https://<your-username>.github.io/project-2037-dashboard/`
7. Open that URL on your phone and add it to your home screen (Safari: Share → Add to Home Screen; Chrome: ⋮ → Add to Home screen). It now behaves like a bookmarked app, same as the Google Sheet does today.

No build step, no server, no ongoing cost — GitHub serves the two static files directly.

## 2. Updating the data

Whenever you want to refresh the dashboard (e.g. after your monthly Google Sheet update):

1. Open your Google Sheet, note the current values from the **Portfolio** and **Dashboard** tabs.
2. On GitHub, open `data.json` in the repo, click the pencil (Edit) icon.
3. Update the relevant fields — mainly:
   - `holdings[]` — one object per row in your Portfolio tab (`current_value`, `current_price`, `gain_loss`, `return_pct`)
   - `monthly_snapshot[]` / `history[]` — append a new entry for the latest month
   - `meta.baseline_date` — bump to the new "as of" date
4. Commit the change (bottom of the edit page — "Commit changes directly to the `main` branch"). GitHub Pages redeploys automatically within ~30–60 seconds.

Because it's plain JSON, this is a copy-paste job — no formulas to break, no scripts to run.

## 3. What the dashboard shows

- **Headline cards**: live total portfolio value, pension corpus vs plan (RAG-coded), Property Engine ISA vs plan, Emergency Shield progress bar, debt status, 2037 net worth target.
- **Cash flow allocation**: where the £1,968 monthly surplus is routed (Emergency Shield vs Property Engine ISA), plus the separate 15% salary-sacrifice pension stream.
- **Pension Engine**: the three named accounts (Aviva active, L&G legacy, Vanguard SIPP legacy) plus a chart comparing your model's year-by-year plan against your actual current value.
- **De-risking glide path**: a checkable year-by-year timeline (2026–2037) sourced from your own milestone table, including the 100%→80/20 equity de-risking from 2033.
- **Scenario toggle**: Chennai vs staying in the UK — switches the highlighted end-state card.
- **Portfolio holdings table**: every row from your Portfolio tab, filterable by owner/account type, searchable.
- **Portfolio value over time**: your History tab plotted as a line chart.
- **Action log**: your immediate next-steps checklist, with state saved locally in the browser.

## 4. Data quality note

Two things worth knowing about the source workbook, carried over as-is (not corrected, since I didn't want to silently rewrite your model):

- The **"Total (in INR)" row** on your Dashboard tab currently mirrors the Stocks Value figure rather than showing a true INR-denominated total — worth checking the formula next time you're in the sheet.
- **RAG comparisons** (e.g. ISA showing 40% of the 2026 plan) are a straight actual-vs-milestone ratio. The Property Engine ISA specifically looks behind because the new £1,516/month contribution only starts September 2026 per your plan — not a real shortfall yet.

## 5. Swing trading (₹10,000 pot)

Per your spec, this is a deliberately separate sandbox from the 2037 architecture and isn't included here — happy to build a companion tracker for it if useful.
