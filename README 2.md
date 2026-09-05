# Job Offer & Equity Calculator

A single-page, client-side calculator for deciding whether to leave a job with unvested equity on the table for a new offer. Enter your current package and a prospective offer, and it models three things side by side: how much equity you'd forfeit by leaving, how long the new offer takes to earn that back, and how commission upside compares between a quota you've already proven out and a quota you haven't.

![Calculator screenshot](assets/screenshot-desktop.png)

Built as one dependency-free `index.html` file — no build step, no backend, no data leaves your browser. Open it locally, host it on GitHub Pages, or add it to your phone's home screen as a lightweight web app.

## Why this exists

Comparing job offers is easy when it's just base salary. It gets harder when:

- You have **unvested equity** that resets to zero (or partially resets) if you leave before a cliff or vesting date.
- The new offer's commission is **quota-based**, but you have no track record against that specific quota yet.
- You're already **outperforming your current quota**, so your realized pay is higher than the "target" numbers on either offer letter.

This tool puts all three effects into one model instead of three separate mental spreadsheets.

## What it calculates

### 1. Cost of leaving before you vest

You enter your total unvested equity value and your vesting cliff (in years), then choose a vesting model:

- **Cliff (all-or-nothing):** you forfeit 100% of unvested equity until the cliff date, then it vests in full.
- **Linear (pro-rata monthly):** the unvested pool vests in equal monthly installments across the cliff period, so leaving partway through means keeping a pro-rata share.

Drag the "months until you'd leave" slider and the chart redraws the vested-vs-forfeited split in real time, with a marker line at your selected exit month.

**Formula:**

```
cliffMonths = cliffYears × 12
vestedFraction =
    cliff model:  (monthsUntilLeave >= cliffMonths) ? 1 : 0
    linear model: min(1, monthsUntilLeave / cliffMonths)
lostEquity = unvestedTotal × (1 − vestedFraction)
```

### 2. Break-even timeline for the new offer

This is the core question: does the new offer's higher pay recover the equity you're walking away from, and if so, when?

The chart plots **net advantage of the new offer vs. staying** — a single line, not two overlapping ones — so the crossover point is obvious: red while you're behind, green once the higher pay has recovered the forfeited equity, with a dashed marker at the exact break-even year.

**Formula:**

```
tdcCurrent = currentBase + (commissionTarget × currentQuota%) + currentEquityAnnual
tdcNew     = newBase + (newBase × newCommissionTargetPct × newQuota%) + newEquityAnnual
delta      = tdcNew − tdcCurrent

breakevenYears =
    lostEquity <= 0        → 0 (nothing to recover)
    delta > 0               → lostEquity / delta
    delta <= 0               → ∞ ("this offer never breaks even")
```

### 3. Quota performance: proven upside vs. new-role uncertainty

Your current quota attainment is a track record — actual, realized performance against a quota you understand. The new role's quota is a guess until you've been in seat for a few cycles. This chart holds the new offer's base and equity fixed and re-runs its total comp across a spread of attainment levels (50% to 200%), plotted against a flat reference line for your current, proven total comp.

This is meant to answer: "if I only hit 70% in my first year while I ramp up, am I still ahead of where I am today?" — rather than only comparing the optimistic, on-target case.

## Inputs

| Field | Side | Description |
|---|---|---|
| Base salary | Current & New | Annual base pay |
| Commission target (100% quota) | Current | Dollar commission at exactly 100% of quota |
| Current quota attainment | Current | Your trailing actual performance, e.g. 165% |
| Equity grant, annual vest value | Current & New | The equity component of TDC — annual vesting value, not total grant size |
| Total unvested equity | Current | The pool at risk if you leave before vesting |
| Vesting cliff (years) | Current | Time until unvested equity would fully vest |
| Vesting model | Current | Cliff (all-or-nothing) or Linear (pro-rata monthly) |
| Target commission, % of base | New | New offer's on-target commission as a percentage of base |
| Assumed quota attainment, new role | New | Your best-guess attainment scenario for the new role |
| Months from today until you'd give notice | Current | Drives the vesting/forfeiture chart marker |

All monetary fields use `inputmode="decimal"` for a proper numeric keypad on iOS; quota/percentage/timing fields are sliders; the vesting model is a segmented toggle. Everything recalculates live on every input change — there's no "Calculate" button and no page reload.

## Assumptions & limitations

Read the footer in the app for the full list, but in short:

- Commission scales **linearly** with attainment — no accelerators, caps, or decelerators are modeled.
- Equity figures are the **annual vest value** you enter, not the total grant size or number of units.
- "Forfeited equity" only reflects the unvested pool you enter — it doesn't model any new equity you'd earn going forward in either role.
- Comp is assumed **flat with no raises** in either scenario, for a clean apples-to-apples comparison.
- This is a personal-planning tool, not tax, legal, or financial advice. Signing bonuses, RSUs vs. options tax treatment, and clawback terms are not modeled — factor those in separately.

## Tech stack

- **Vanilla HTML/CSS/JavaScript** — no framework, no build step, no npm install required to run it.
- **[Chart.js 4.4.4](https://www.chartjs.org/)** via CDN for the three charts, with a couple of custom plugins (`afterDatasetsDraw` hooks) for the exit-month and break-even markers.
- **[Fontshare](https://www.fontshare.com/)** for General Sans (display) and Satoshi (body); **Google Fonts** for JetBrains Mono (all numeric values, tabular figures).
- Light/dark mode via a `data-theme` attribute and CSS variables, toggled with a header button — no `localStorage` is used, so the theme choice doesn't persist across reloads.
- No backend, no analytics, no external calls other than the CDN assets above. Nothing you type is transmitted anywhere.

## Running it locally

It's a static file — any static server works:

```bash
npx serve .
# or
python3 -m http.server 8000
```

Then open the printed URL in your browser. You can also just double-click `index.html` to open it directly from the filesystem, though a local server avoids any browser quirks with `file://` URLs.

## Hosting on GitHub Pages

1. Add `index.html` to your repository — either at the repo root or inside a `/docs` folder.
2. Commit and push.
3. In the repo's **Settings → Pages**, set the source to the branch and folder where the file lives, then save.
4. GitHub will publish it at `https://<username>.github.io/<repo>/` within a minute or two.

## Adding it to your iPhone home screen

The file already includes iOS home-screen meta tags (`apple-mobile-web-app-capable`, `apple-mobile-web-app-title`, `viewport-fit=cover`, a themed status bar, and an inline SVG favicon), so once it's hosted:

1. Open the GitHub Pages URL in **Safari**.
2. Tap **Share → Add to Home Screen**.

It launches full-screen without Safari's address bar, so it behaves like a native app.

## Customizing it for your own numbers

Everything is driven by the input fields — nothing is hardcoded except the default values that pre-populate the form. To change your own defaults, open `index.html`, search for the `value="..."` attributes on the input elements in the HTML, and replace them with your own base/commission/equity figures.

## License

No license file is included by default — add one (MIT is a common choice for a small personal tool like this) if you plan to make the repository public and want to clarify reuse terms.
