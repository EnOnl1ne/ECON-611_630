# Plan: SGIP HFTD Eligibility & Battery Storage Funding Outcomes

## Context
ECON 676 (Natural Resource Economics and Development Policy) proposal, due December. The
professor requires a causal design with a prima facie (preliminary) result, not just a
proposal. Findings are also meant to be useful to Ava Community Energy's VPP and battery storage programs,
since I intern for them.

## Research question
For residential Self-Generation Incentive Program applicants in PG&E territory (which also bounds Ava's territory), 
does being located in a High Fire-Threat District (HFTD Tier 2/3) — which qualifies a household for
enhanced SGIP incentive levels — predict a higher probability of reaching funded (`Paid`)
status, relative to applicants not in an HFTD?

Extended, deferred work not for this first pass, but for later examination: at the zip-code level, do
zips located entirely within HFTD Tier 2/3 show a higher rate of SGIP applications per 1,000
housing units than zips with no HFTD designation? Requires an ACS housing-unit merge — see
"Deferred work" below.

## Identification strategy
Similar to, but not quite a regression discontinuity with a continuous running variable — HFTD tier is a
discrete, administratively-assigned category (Tier 2 / Tier 3 / Not Applicable), not a
percentile score. This creates a discontinuity by eligibility category, so we can estimate with zip-code fixed effects
to net out neighborhood-level confounds, using the subset of zip codes that contain both Tier 2/3 and Not Applicable
applicants.

## Data files (all in ECON 611_630/SGIP_HFTD, read-only — copy before modifying)

**Primary file — the one actually used for analysis:**
- `Weekly_Statewide_Report_09_20_2026_posted_09_20_26_.xlsx` (about 106,500 rows). This is the
  SGIP-specific application-level dataset (downloaded from
  https://www.selfgenca.com/documents/reports/statewide_projects). It contains eligibility,
  funding status, and outcome fields — this is the only file with the actual treatment and
  outcome variables.
  - Two sheets: `SGIP Statewide Report <date>` (data) and `Report Field Tooltips` (a partial
    data dictionary — not exhaustive; confirm fields against the real header row, not just
    the tooltip sheet).
  - **Header row is row 4, not row 1.** Rows 1–3 are a title/date stamp and a blank row.
  - Key fields confirmed present and populated: `SGIP Administrator`, `Budget Category`,
    `Budget Classification`, `Fully Qualified State`, `Host Customer Sector`, `Equipment Type`,
    `Program Year`, `Located in HFTD (Tier 2, Tier 3)`, `Experienced Two PSPS Events
    (True/False)`, `Energy Storage Capacity (kWh)`, `Zip`, `City`, `County`.

**Reference-only files — NOT the right data for this question, keep for context/citations:**
- `IMG_6673.jpeg`, `IMG_6674.jpeg` — course assignment-sheet photos (paper proposal
  requirements: ~10–15 pages, 20% of grade, must identify an open problem, a research
  question, and a research-based means of answering it, with graduate students expected to
  bring a prima facie result). Reference for what the proposal document itself needs to
  contain — not analysis input.

**Extension file (deferred, not yet downloaded):** ACS 5-year estimates, Table B25001 (Total
Housing Units), ZCTA level. Needed only for the secondary zip-level adoption-rate question.
Confirm SGIP zip codes map cleanly to ZCTAs before merging — zip codes and ZCTAs are related
but not identical geographies.

## Sample construction
Apply in this order, on the primary file:
1. `SGIP Administrator` == `Pacific Gas and Electric`
2. `Host Customer Sector` in `{Residential, Single Family, Multifamily}`
3. `Energy Storage Capacity (kWh)` is not null (restricts to storage projects)
4. Exclude `Fully Qualified State` == `RRF Rejected` (these never received a Budget Category,
    just applications that never progressed far enough in review;
   see "Known data issues" below)
5. `Program Year` >= 2024 (post-AB-209 / CPUC Decision 24-03-071, which consolidated the old
   separate Equity and Equity Resiliency budgets into one unified residential equity budget —
   mixing pre/post regimes would conflate two different eligibility structures in the same
   sample)

**Verified against the actual file on 2026-10-02**: PG&E rows 47,930 → after residential-sector filter
43,466 → after storage-not-null filter 43,400 → after excluding 1,873 `RRF Rejected` rows
41,527 → after `Program Year >= 2024` filter **9,910 total**, split as **1,771 Tier 2 / 780
Tier 3 / 6,832 Not Applicable** (527 rows have no HFTD value recorded at all in this cut).
These counts matched the plan's earlier estimate exactly, so the filter logic above is
confirmed correct on this copy of the file.

The full post-2017, pre-year-filter sample is larger (9,939 Tier 2 / 4,023 Tier 3 / 16,371 Not
Applicable, 11,194 with no HFTD value) if more power is needed and the AB-209 regime change is
instead handled with a year fixed effect — treat that as a robustness check, not the primary
spec.

## Known data issues to carry forward
- **`Budget Category` is blank for ~5% of PG&E residential storage rows.** Already diagnosed:
  ~83% of blanks are `RRF Rejected` applications (excluded by filter 4 above); the small
  remainder is a 2011–2016 legacy cohort predating the current budget taxonomy. Not a threat
  to the design once rejected applications are excluded — confirm this on re-derivation rather
  than assuming it still holds after any file update.
- **`Located in HFTD` stays populated even where `Budget Category` is blank** (~74% of those
  blank-Budget-Category rows had a real HFTD value, mostly `Not Applicable`, which is itself
  informative, not missing). This is why `Located in HFTD` — not `Budget Category` — is the
  field actually used as the treatment indicator.
- Zip code, not address/lat-long, is the finest geography in this file — no literal
  distance-to-boundary running variable is possible. This is why the design uses zip fixed
  effects rather than a bandwidth-based RD.
- `Fully Qualified State` and `Budget Classification` both carry the funding/application-status
  information (`Waitlist`, `Reserved`, `Paid`, `Pending Reservation`, `Cancelled`, `PBI in
  Process`) — use `Budget Classification` for the `Paid` outcome indicator, since it's the
  more granular of the two for this purpose.

## Analysis steps
1. **Load and clean**: read the primary file with header at row 4, apply the sample
   construction filters above in order, logging the row count remaining after each filter.
2. **Descriptives**: cross-tab `Budget Classification` by `Located in HFTD` on the final
   cleaned sample. **Verified on 2026-10-02, post-2024 sample:** `Paid` rate is 50.1% for Tier
   2 (n=1,771), 44.4% for Tier 3 (n=780), and 33.9% for Not Applicable (n=6,832) — a real,
   fairly large gap, and notably Tier 2 outperforms Tier 3, which is worth explaining in the
   write-up rather than only reporting the pooled Tier 2/3 estimate. (An earlier,
   less-restricted cut of this data — not yet filtered to 2024+ — had shown ~65% vs. ~54%;
   that number is superseded by the figures above and should not be reused.)
3. **Identify zip codes with both tier statuses** (both a Tier 2/3 record and a Not Applicable
   record) on the final cleaned sample — this is the fixed-effects identifying variation.
   **Verified on 2026-10-02, post-2024 sample:** 357 zips have any Tier 2/3 record, 598 have
   any Not Applicable record, and **238 zips have both** — this 238 is the fixed-effects
   identifying sample size to use, not a number from an earlier, differently-filtered run.
4. **Main specification**: logistic regression (report a linear probability model alongside it
   for interpretability) of `Paid` (binary, derived from `Budget Classification`) on `Located
   in HFTD` (Tier 2/3 vs. Not Applicable), with zip fixed effects. Cluster standard errors by
   zip.
5. **Robustness checks**:
   - Same spec without zip fixed effects, to show how much the estimate moves
   - Tier 2 and Tier 3 as separate indicators rather than pooled
   - Full post-2017 sample with year fixed effects instead of the 2024+ restriction, as a
     sensitivity check on the AB-209 regime-change handling
   - Include `Experienced Two PSPS Events` as an additional covariate / alternative treatment
     definition
6. **Output**: a results table (coefficients, SEs, N, number of zip clusters) and a short
   written summary of direction, magnitude, and significance — written at the level of
   confidence the design actually supports (a discontinuity-in-eligibility-category estimate
   with zip fixed effects, not a textbook sharp RD).

## Deferred work (explicitly out of scope for this pass)
- ACS housing-unit merge for the zip-level adoption-rate question (secondary research question
  above)
- GIS-based treatment-intensity weighting using CPUC HFTD shapefiles overlaid on ZCTA
  boundaries (would allow using zips with mixed tier status by weighting rather than requiring
  a clean split)
- Stockton/Lathrop NEM 3.0 difference-in-differences idea — a separate project exploring
  whether Ava's territorial expansion changed solar/storage adoption. Needs the actual DGStats
  Interconnected Applications *data* file downloaded (only the data dictionaries are on hand
  so far), plus resolution of known comparability issues between Stockton and Lathrop
  (Lathrop's ~49% population growth since 2020 likely means much of its solar is
  builder-installed under California's new-construction solar mandate, not a household choice
  comparable to Stockton's).

## Deliverable
A cleaned analysis dataset, the regression results table, and a short write-up connecting the
funding-outcome finding to Ava's battery fleet/VPP outreach targeting: if eligibility predicts
funding success, that supports targeting outreach by HFTD geography; if not, it suggests
awareness or application complexity — not incentive generosity — is the binding constraint,
which would call for a different outreach approach.
