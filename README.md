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
- **Net Worth Glide Path**: target net worth vs. what you're actually on track for at your *current* ISA contribution rate — the single chart that answers "am I on schedule."
- **Property Engine — Plan vs. Reality**: three lines (full plan rate, your currently confirmed rate, recorded actual) plus two contribution-rate meters, so a contribution shortfall is visible immediately rather than buried in a milestone table.
- **Margin of Safety**: a RAG-rated inventory of every shock-absorber in the plan (Emergency Shield, de-risking, insurance, dual-path optionality, etc.), a stress test showing what a −35% equity shock in 2036 would do to the pension and ISA specifically, and a prioritised list of what closes the gap. All of this reads from `glide_path` in `data.json` — see section 6 below for how to keep it current.
- **Scenario toggle**: Chennai vs staying in the UK — switches the highlighted end-state card.
- **Portfolio holdings table**: every row from your Portfolio tab, filterable by owner/account type, searchable. Rows using a stale/floored price (see section 5) are marked with a ⚠ next to the value — hover it for why.
- **Portfolio value over time**: your History tab plotted as a line chart.
- **Action log**: your immediate next-steps checklist, with state saved locally in the browser.

## 4. Data quality note

Two things worth knowing about the source workbook, carried over as-is (not corrected, since I didn't want to silently rewrite your model):

- The **"Total (in INR)" row** on your Dashboard tab currently mirrors the Stocks Value figure rather than showing a true INR-denominated total — worth checking the formula next time you're in the sheet.
- **RAG comparisons** (e.g. ISA showing 40% of the 2026 plan) are a straight actual-vs-milestone ratio. The Property Engine ISA specifically looks behind because the new £1,516/month contribution only starts September 2026 per your plan — not a real shortfall yet.

## 5. Two bugs found and fixed on 2026-09-27

**A. Four headline cards were silently showing £0.** `index.html` filtered pension/ISA/cash holdings on `account_type` (which holds specific names like `"Aviva Pension"` or `"Stocks ISA"`), not `category` (which holds the roll-up `"Pension"`/`"ISA"`/`"Cash"` that these cards actually need). Pension Corpus, Property Engine ISA, and the combined Emergency Shield card were all filtering on a value that never matches, and had been showing £0 regardless of the real portfolio value. Fixed by switching those four filters to `category`. Worth double-checking any other page that reads this same `data.json` for the same mix-up.

**B. Several holdings had corrupted or missing live prices.** Two classes of problem in the `data.json` your Apps Script last published:

- `current_price: null` for platform-internal pension fund names and the SGLN gold ETC — GOOGLEFINANCE simply can't resolve these, and the script was zeroing the value rather than falling back to something sane.
- A handful of other holdings had a **mismapped price** (an Instruments-tab crosswire, the same failure class flagged before in the dashboard automation log) — e.g. a bond fund and a pension equity fund sharing the same `current_price`, producing implausible returns like +6086% or −98.9% instead of a real number.

Both are now handled the same way: any holding whose price is missing, or whose implied return falls outside a plausible band for its asset class (equity −50%/+150%, bond −20%/+30%, commodity −30%/+50% — cash is exempted since its "return" is small-principal interest accumulation, already verified correct), is **floored at cost basis** and flagged `"price_stale": true` with a `price_stale_note` explaining why. This is deliberately conservative — a floor, not a fabricated live value — and it's why some totals in this update are lower than they'd be if the corrupted prices were trusted at face value. The flagged rows are marked ⚠ in the holdings table.

**This needs a fix at the source, or it will recur on the next auto-publish.** Code.gs currently has no fallback for a failed or implausible GOOGLEFINANCE lookup — the fix belongs there (or in the Sheet, as a manual price column for the handful of funds GOOGLEFINANCE can never resolve), not just in this one commit. Added as an item to the Action Log on the dashboard itself so it doesn't get lost; see `meta.data_quality_note` in `data.json` for the exact list of affected holdings.

## 6. Keeping the Glide Path / Margin of Safety section current

These live in a new `glide_path` key in `data.json`:

- `isa_target_planrate_gbp` / `isa_target_currentrate_gbp` / `networth_target_gbp` / `networth_currentrate_gbp` — year-keyed target curves. Only regenerate these if the underlying assumptions change (contribution amounts, growth rate, baseline). They were built to land within ~1.5% of this project's own stated 2037 targets.
- `contribution_rates` — the two `*_actual_monthly_gbp` fields are the only ones you should need to touch regularly: update them whenever your real standing order changes, and the Property Engine chart and meters recompute from them.
- `stress_test`, `buffers`, `recommendations` — qualitative/scenario content, worth revisiting every few months or after a real market move, not on every data refresh.

## 5. Swing trading (₹10,000 pot)

Per your spec, this is a deliberately separate sandbox from the 2037 architecture and isn't included here — happy to build a companion tracker for it if useful.
