# fed-voting

Single-file interactive chart: every FOMC dissenting vote, Jan 2016–Jul 2026, plotted
against the fed funds target path. Tests whether the size/direction of the dissent
bloc at a meeting predicts the committee's next rate decision.

**Live at:** https://rjre.github.io/fed-voting/ (GitHub Pages, served from `main`, root).

There is exactly one file that matters: **`index.html`**. Plain HTML/CSS/vanilla JS,
hand-rolled SVG, no chart library, no framework, no build step. Do not introduce one.
The entire dataset (87 meetings) is embedded as a JSON literal (`const MEETINGS = [...]`)
inside the `<script>` tag — there is no separate data file and no fetch() call.

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

## Design spec (institutional tally sheet)

Fixed palette, no gradients, no rounded corners, no dark-mode variant (this is a
single deliberate print-like aesthetic, not a themeable dashboard):

| Token | Hex | Use |
|---|---|---|
| `--bg` | `#EDEFEE` | page background |
| `--ink` | `#14181C` | text, lines, borders |
| `--tighter` | `#7A2B2B` | dissent for tighter policy |
| `--easier` | `#2E5C7A` | dissent for easier policy |
| `--other` | `#7A6A32` | dissent on other/procedural grounds |
| `--assent` | `#A8AFAC` | assenting majority baseline cell |
| `--hair` | `#C9CFCB` | hairline gridlines |

Fonts (Google Fonts, loaded via `<link>`, no self-hosting): **Archivo Narrow 700**
for labels/headers, **Spectral** for body prose, **IBM Plex Mono** for figures/
dates/tabular numbers. This palette was run through the dataviz skill's
`validate_palette.js` — it fails the formal lightness/chroma/contrast checks
because it's a deliberately muted, desaturated "institutional" palette rather
than a high-chroma categorical scheme. Compensated with secondary encoding per
the skill's own escape hatch: every cell has a 0.6px ink stroke (so the low-
contrast assent grey stays legible against the background), and identity never
depends on color alone — the hover/focus readout always spells out each
dissenter's name and classification in text, and the full data table repeats it
with a colored-dot + text label. Don't just re-run the validator and "fix" the
hexes — the muted look is the brief.

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
