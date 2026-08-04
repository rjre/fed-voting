# fed-voting

Single-file interactive chart: every FOMC dissenting vote, Jan 2016–Jul 2026, plotted
against the fed funds target path. Tests whether the size/direction of the dissent
bloc at a meeting predicts the committee's next rate decision.

**Live at:** https://rjre.github.io/fed-voting/ (GitHub Pages, served from `main`, root).

There is exactly one file that matters: **`index.html`**. Plain HTML/CSS/vanilla JS,
hand-rolled SVG, no chart library, no framework, no build step. Do not introduce one.
The entire dataset (87 meetings) is embedded as a JSON literal (`const MEETINGS = [...]`)
inside the `<script>` tag — there is no separate data file and no fetch() call.

## Workflow standing instructions

- **Commit and push straight to `main`.** No feature-branch/PR workflow for this repo —
  the user asked for this explicitly. (An earlier `claude/fomc-dissent-chart-...`
  branch exists from before that instruction; it's stale, ignore it.)
- **Formatting standard is ruffer.co.uk**, not an invented palette (see below) — if the
  visual design ever needs revisiting, re-derive it from Ruffer's own site rather than
  from taste or a generic dataviz default.

## Data provenance — how the dataset was built

Three sources, cross-checked against each other:

1. **St. Louis Fed dissent tracker** — authoritative record of dissent counts,
   chair, total voters, votes for/against, and each dissent's tighter/easier/other
   classification (the file already classifies relative to the action taken, which
   is why its classification was trusted as-is rather than re-derived).
   - Page: https://www.stlouisfed.org/on-the-economy/2014/september/a-history-of-fomc-dissents
   - File: https://www.stlouisfed.org/-/media/project/frbstl/stlouisfed/files/excel/fomc_dissents_data.xlsx
   - Last updated (per file's own header row): **7/31/2026**. Re-download this file
     for any future update — don't hand-edit dissent counts.
2. **FRED series DFEDTARU / DFEDTARL** (daily target range upper/lower bound) —
   used to derive the actual action size (bp) and prior/new range at each meeting.
   Every FOMC meeting date in the xlsx was matched to the nearest target-range
   change date within 0–3 days (the new range typically takes effect the day after
   a 2-day meeting concludes). All 31 post-2015-12-16 rate changes matched exactly
   one meeting each — no orphans, no ambiguity.
   - https://fred.stlouisfed.org/series/DFEDTARU
   - https://fred.stlouisfed.org/series/DFEDTARL
3. **federalreserve.gov FOMC statements** — one per meeting, URL pattern
   `https://www.federalreserve.gov/newsevents/pressreleases/monetary{YYYYMMDD}a.htm`
   (verified against multiple real fetches spanning 2016–2026, including the two
   March 2020 emergency meetings and 2026 meetings under Warsh — pattern holds
   throughout). This is the `source` field on every row in `MEETINGS`, and is
   quoted (each dissenter's stated preference, `quote` field) for all 27 meetings
   that had at least one dissent. Full names (`full` field) were also pulled from
   these statements since the xlsx only records surnames.

Every `MEETINGS[i]` row therefore carries its own citation (`source`) — there is no
row without one, per the original brief.

### Known real-world facts baked into the data (verify if this ever looks wrong)

- Kevin Warsh was sworn in as Fed Chair **May 22, 2026**, succeeding Powell; the
  FOMC unanimously elected him committee chairman the same day. First meeting
  under his chairmanship in the dataset is **2026-06-17**. Confirmed via
  federalreserve.gov press release `other20260522a.htm` and contemporaneous news
  coverage (CNBC, CNN, NPR, Al Jazeera) — this is real, not a hallucination, in
  case a future session's training cutoff predates it and it looks suspicious.
- Rate held at 3.50–3.75% from the 2025-12-10 cut through the end of the sample
  (2026-07-29 meeting) — five consecutive holds in 2026 despite a growing dissent
  bloc (Miran, then Hammack/Kashkari/Logan in the opposite direction in July).

### Regenerating / extending the dataset

To add meetings after July 2026:
1. Re-download `fomc_dissents_data.xlsx` (link above) — it's a living document,
   "regularly updated."
2. Pull new DFEDTARU/DFEDTARL observations from FRED for the new date range.
3. Match new change-dates to meetings (±0–3 days), compute `action_bp` as the
   upper-bound delta.
4. For any meeting with a dissent, fetch its statement at the URL pattern above
   and pull the exact vote line for `quote` and `full` name — don't paraphrase;
   the point of a `quote` field is that it's verbatim.
5. Append to the `MEETINGS` array in `index.html`. All derived stats
   (`net_hawkish`, the conditional/base hike probabilities, mean-dissents
   comparison, hawkish-vs-hold case table) are computed live in the page's own
   `<script>` from `MEETINGS` — nothing else needs updating by hand.

## Methodology notes

- **Classification is relative to the action taken, not the prior setting.** E.g.
  Loretta Mester dissenting on 2020-03-15 (committee cut 100bp to 0–0.25%) is
  classified **tighter** because she preferred a smaller cut (to 0.50–0.75%), even
  though her preferred rate was still a cut from the prior meeting. Verified
  against the actual statement text for every one of the 27 dissent meetings.
- **Total voters vary 9–12** across the sample due to governor vacancies — never
  assumed to be 12. Absent members (e.g. Kugler on 2025-07-30) are excluded from
  the voter count, matching the xlsx's own `FOMC Votes` figure.
- The ballot grid draws one **assent** cell (grey) per meeting as a baseline plus
  one stacked cell per dissenting vote — not "one cell per voter" — per the
  original chart spec ("one cell per dissenting vote").

## Computed findings (reproduced live in the page, under "What the dissents say
about the next move" — these are not hard-coded, they're computed by the page's
own JS from `MEETINGS`; the numbers below are what that computation currently
produces and were reported to the user before any styling was done)

- **P(hike at t+1 | net hawkish dissent at t) = 15.4%** (n=13 meetings with more
  tighter than easier dissents) **vs. P(hike at t+1) unconditional base rate =
  22.1%** (n=86). Counterintuitively *lower*, not higher — hawkish dissents
  cluster ahead of a long run of holds (2016 George dissents preceded five holds
  before the December 2016 hike) more often than they precede an imminent hike.
  Sample is small (n=13); read this as suggestive, not conclusive.
- **Mean dissents at meetings where the rate moved: 0.73** (n=30, 22 total
  dissenting votes) **vs. meetings where it held: 0.42** (n=57, 24 total
  dissenting votes). Inflection points generate more disagreement than holds do.
- **Hawkish (tighter) dissent cast against a hold, with meetings until the next
  actual change:** six cases —
  - 2016-03-16 (George) → next change 2016-12-14, 6 meetings later
  - 2016-04-27 (George) → 2016-12-14, 5 meetings later
  - 2016-07-27 (George) → 2016-12-14, 3 meetings later
  - 2016-09-21 (George, Mester, Rosengren) → 2016-12-14, 2 meetings later
  - 2016-11-02 (George, Mester) → 2016-12-14, 1 meeting later
  - 2026-07-29 (Hammack, Kashkari, Logan) → no change through end of sample

## Design spec — ruffer.co.uk house style

The page was first built with an invented "institutional tally sheet" palette
(parchment background, Archivo Narrow / Spectral / IBM Plex Mono). The user then
asked for **ruffer.co.uk's actual formatting standards** instead, so the chrome was
re-derived from Ruffer's real stylesheet and chart component, not from taste. If
this is ever redone, re-pull from Ruffer rather than reusing the table below from
memory — their site can change.

How it was derived (repeatable): fetch `https://www.ruffer.co.uk`, find its main
CSS bundle link (`/bundles/styles?v=...`) plus `/assets/css/rf-graph.css` (their
own chart component), download both, and grep for `font-family`, hex colors, and
`h1`/`h2`/`body` rules. That's what produced the table below — it's read directly
off their production CSS, not guessed.

No gradients, no rounded corners (kept from the original brief — Ruffer's own site
uses a little border-radius on buttons, but square held here), no dark-mode variant.

| Token | Hex | Use | Source |
|---|---|---|---|
| `--bg` | `#ffffff` | page background | Ruffer `body{background-color:#fff}` |
| `--panel` | `#f8f8f8` | card/panel background | Ruffer `.grey-panel{background:#F8F8F8}` |
| `--ink` | `#383838` | body text | Ruffer `body{color:#383838}` |
| `--heading` | `#086132` | headings, rate line, links-on-hover | Ruffer `.h1,.h2{color:#086132}` |
| `--accent` | `#4e9a33` | default link color | Ruffer `a{color:#4e9a33}` (most-used hex in their bundle after white) |
| `--muted` | `#797979` | axis labels, captions, secondary text | Ruffer's own `rf-graph.css` axis-tick color |
| `--hair` | `#e5e5e5` (chart gridlines `#e9e9e9`) | hairlines/rules | close to Ruffer's `path.domain{stroke:#E9E9E9}` |
| `--tighter` | `#7A2B2B` | dissent for tighter policy | kept from the original brief — not a Ruffer color, chosen to stay legible and distinct from the brand green |
| `--easier` | `#2E5C7A` | dissent for easier policy | ditto |
| `--other` | `#8a6d1f` | dissent on other/procedural grounds | ditto |
| `--assent` | `#b3b3b3` | assenting majority baseline cell | swapped to a grey Ruffer actually uses |

**Fonts — no webfonts fetched at all**: **Georgia** (system font, present on every
OS) for headings and body copy, matching Ruffer's `body{font-family:"Georgia W01
Regular"}`/`.h1{font-family:"Georgia W01 Regular"}` (their Georgia is a paid
Monotype cut loaded from fonts.net — not something to license or hot-link, so
plain system Georgia stands in). A system sans stack — `'Avenir Next', Avenir,
'Segoe UI', Roboto, Helvetica, Arial, sans-serif` — stands in for their self-hosted
Avenir LT W01 (same reasoning) for labels, nav-style chrome, axis text, and table
headers. Figures/dates use a plain `ui-monospace` stack. Headings are **not**
bold/uppercase — Ruffer's `.h1`/`.h2` are `font-weight:normal`; only small
label-style text (legend, table headers, tags) is uppercase+bold, in the sans
stack, mirroring how Ruffer treats its own nav/small-print chrome vs. its
Georgia editorial voice.

The categorical dissent colors (tighter/easier/other/assent) were deliberately
**not** replaced with Ruffer greens — using the brand color for one dissent
category would read as "this is what the brand means," which isn't the point of
a classification channel. Only the chrome (backgrounds, rules, headings, body
text, links, axis ink) was pulled from Ruffer; the semantic/categorical encoding
stayed put. Contrast/legibility compensation (thin borders on every ballot cell,
text always spelling out the classification alongside color, per the dataviz
skill's escape hatch for a muted palette) is unchanged from the original build.

The stepped rate line draws in on load (`stroke-dasharray`/`stroke-dashoffset`
animated via `getTotalLength()`, `prefers-reduced-motion` respected) — lifted
from Ruffer's own `rf-graph.css` `@keyframes rf-graph-dash`, same technique.

## Chart mechanics

- Upper panel: stepped path (`M`/`L` only, no curves) of the target range **upper
  bound**, in an inline SVG built by JS from `MEETINGS` (`xScale`/`yScaleU` in the
  script). Continuous date x-axis, not categorical — meetings that fall close
  together (e.g. the three March 2020 emergency meetings) are genuinely close
  together on screen, not artificially evened out.
- Gap strip between panels: small up/down triangle ticks at every meeting where
  `action_bp !== 0`.
- Lower panel: ballot grid, one column per meeting at its true x position, one
  grey assent cell plus one stacked cell per dissenting vote (colored by class),
  2px gaps between stacked cells.
- Hover **or keyboard focus** (each meeting has an invisible, tab-focusable hit
  rect wider than its visual column) shows a crosshair + highlighted column +
  updates a fixed readout panel below the chart that reproduces that meeting's
  vote line verbatim, with a link to the statement. Hover and focus are wired to
  the same handler so keyboard users get identical detail.
- A `<details>` element holds the full 87-row data table (with per-row source
  links) as the non-interactive fallback / citation trail.

## Verifying changes

No build step, so "testing" means opening the file in a browser and checking the
console. There's no project-specific run skill yet; ad hoc verification used a
local `python3 -m http.server` plus Playwright driving the pre-installed
Chromium at `/opt/pw-browsers/chromium` (note: pass `executablePath:
'/opt/pw-browsers/chromium'` — that's the symlink itself, not a directory).
Check for console errors, screenshot full-page, and screenshot after hovering/
focusing a hit column to confirm the readout updates.
