# fed-voting

Single-file interactive chart: every FOMC dissenting vote, Jan 2016–Jul 2026, plotted
against the fed funds target path. Tests whether the size/direction of the dissent
bloc at a meeting predicts the committee's next rate decision.

**Live at:** https://rjre.github.io/fed-voting/ (GitHub Pages, served from `main`, root).

## What this is

There is exactly one file that matters: **`index.html`**. Plain HTML/CSS/vanilla JS,
hand-rolled SVG, no chart library, no framework, no build step. The entire dataset
(87 meetings) is embedded as a JSON literal (`const MEETINGS = [...]`) inside the
`<script>` tag — there is no separate data file and no `fetch()` call.

The chart has two stacked panels sharing a continuous date x-axis:

- **Upper panel** — a stepped path of the fed funds target range upper bound.
- **Lower panel** — a ballot grid: one grey "assent" cell per meeting plus one
  stacked, colored cell per dissenting vote (tighter / easier / other).

Hovering or keyboard-focusing a meeting column highlights it with a crosshair and
updates a readout panel with that meeting's verbatim vote line and a link to the
source statement. A `<details>` element holds the full 87-row data table with
per-row citations as a non-interactive fallback.

## Data provenance

Three sources, cross-checked against each other:

1. **[St. Louis Fed dissent tracker](https://www.stlouisfed.org/on-the-economy/2014/september/a-history-of-fomc-dissents)**
   ([xlsx](https://www.stlouisfed.org/-/media/project/frbstl/stlouisfed/files/excel/fomc_dissents_data.xlsx)) —
   authoritative record of dissent counts, chair, total voters, votes for/against,
   and each dissent's tighter/easier/other classification.
2. **FRED series [DFEDTARU](https://fred.stlouisfed.org/series/DFEDTARU) /
   [DFEDTARL](https://fred.stlouisfed.org/series/DFEDTARL)** (daily target range
   upper/lower bound) — used to derive the actual action size (bp) and prior/new
   range at each meeting.
3. **federalreserve.gov FOMC statements**, one per meeting
   (`https://www.federalreserve.gov/newsevents/pressreleases/monetary{YYYYMMDD}a.htm`) —
   this is the `source` field on every row in `MEETINGS`, and is quoted (`quote`
   field) for all 27 meetings with at least one dissent.

Every row in `MEETINGS` carries its own citation. See `CLAUDE.md` for the full
provenance notes, methodology caveats, and instructions for extending the dataset
past July 2026.

## Findings

Computed live in the page (under "What the dissents say about the next move"),
from `MEETINGS` — nothing is hard-coded:

- P(hike at t+1 | net hawkish dissent at t) = 15.4% vs. unconditional base rate of
  22.1% — hawkish dissents cluster ahead of holds more often than they precede an
  imminent hike (small sample, n=13).
- Mean dissents at meetings where the rate moved: 0.73, vs. 0.42 at meetings that
  held — inflection points generate more disagreement than holds do.

## Verifying changes

No build step, so "testing" means opening `index.html` in a browser and checking
the console:

```
python3 -m http.server
```

Then check for console errors, and confirm hovering/focusing a column updates the
readout panel. See `CLAUDE.md` for the full verification workflow (Playwright
against the pre-installed Chromium).

## Contributing

See `CLAUDE.md` for the design spec (fixed institutional-tally-sheet palette,
fonts), chart mechanics, and step-by-step instructions for regenerating or
extending the dataset.
