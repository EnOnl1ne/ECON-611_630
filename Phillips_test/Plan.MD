Using UNRATE.csv and CPIAUCSL.csv,
test the results of the Phillips curve over time.

# Phillips Curve Test — Plan

## Question
Does the U.S. data show a Phillips curve, meaning a negative relationship between
inflation and unemployment? If it does, has that relationship held steady since 1948
or changed over time?

Three versions to test:
1. **Original Phillips curve:** inflation level vs. unemployment rate.
2. **Expectations-augmented (accelerationist) curve:** *change* in inflation vs.
   unemployment. With adaptive expectations, last year's inflation stands in for
   expected inflation.
3. **Stability over time:** does the slope change across monetary-policy eras?

## Data (FRED, monthly, in the parent folder)
| File | Series | Coverage | Use |
|---|---|---|---|
| `CPIAUCSL.csv` | CPI, all urban consumers, SA (index) | 1947-01 → 2026-08 | Build inflation (y) |
| `UNRATE.csv` | Civilian unemployment rate, % | 1948-01 → 2026-08 | Unemployment (x) |

Paths: the notebook sits in `Phillips_test/`, so read the files with `'../UNRATE.csv'` and `'../CPIAUCSL.csv'`.

## Steps

### 1. Load and merge
- `pd.read_csv(..., parse_dates=['observation_date'], index_col='observation_date')`
  for each file.
- Join into one monthly DataFrame, `df`, with an inner join on the date index.
  The overlap starts in 1948-01.
- Check: `df.info()`, the first and last dates, and no missing values.

### 2. Build the variables
- `infl` = year-over-year CPI inflation, % = `df['CPIAUCSL'].pct_change(12) * 100`.
  (Year-over-year avoids the noise and seasonality of annualized monthly changes.)
- `d_infl` = change in inflation over 12 months = `infl - infl.shift(12)`.
- `unrate_lag` = unemployment 12 months earlier (optional, to test a lagged effect).
- Drop the NaN rows created by the lags. The sample starts around 1949.
- Annual version: `df.resample('YE').mean()` over complete years only (1949–2025).
  Annual points make the scatter plots easier to read.

### 3. Assign eras
Add an `era` column with `pd.cut` on the year:
| Era | Years | Why |
|---|---|---|
| Early postwar | 1949–1969 | Phillips (1958) and Samuelson–Solow (1960) era |
| Great Inflation | 1970–1984 | Oil shocks, Volcker disinflation; the curve "breaks" |
| Great Moderation | 1985–2007 | Low, stable inflation |
| Post-GFC | 2008–2019 | "Flat" Phillips curve, missing disinflation |
| Pandemic & after | 2020–2026 | Unemployment spike, then the 2021–23 inflation surge |

### 4. Describe the data
- Summary stats (mean and std of `infl` and `UNRATE`) by era:
  `df.groupby('era')[['infl','UNRATE']].agg(['mean','std'])`.
- Correlation of `infl` and `UNRATE`, for the full sample and within each era.
- Rolling 10-year correlation (`rolling(120).corr()`) to see how the
  relationship drifts over time.

### 5. Figures (title, axis labels, legend and source note on each)
1. **Time series:** inflation and unemployment on one set of axes since 1949.
2. **Full-sample scatter:** annual `infl` vs. `UNRATE` with a fitted line. The
   expected result is a weak or no relationship.
3. **Scatter by era:** points colored by era, one fitted line per era. This is the
   key figure. Optionally use `plt.subplots(1, 5, sharey=True)` for small multiples.
4. **Expectations-augmented scatter:** `d_infl` vs. `UNRATE` with a fitted line.
   The x-intercept is a rough estimate of the NAIRU.
5. **Rolling correlation over time:** a line plot of the 10-year rolling correlation.
6. *(Optional)* Connected scatter (a "Phillips loop") for the 1970s or 2019–2026,
   with the years labeled via `ax.annotate`.

### 6. Estimate
Use simple OLS with `np.polyfit(x, y, 1)`, or `statsmodels` if it is available.
- Model A: `infl_t = a + b * UNRATE_t`, for the full sample and for each era.
- Model B: `d_infl_t = a + b * UNRATE_t`, for the full sample and for each era.
- Implied NAIRU from Model B: `u* = -a / b`.
- Put the results in one table: era | slope b | intercept a | R² | N | implied u*.
- Prediction: b < 0 in most eras, with the size of b shrinking (the curve
  flattening) after 1985, and Model B fitting the 1970s better than Model A.

### 7. Robustness (pick 1–2)
- Monthly vs. annual data: do the slopes agree?
- Drop 2020-04 → 2020-12 (the COVID unemployment spike is a big outlier).
- Use lagged unemployment (`unrate_lag`) instead of current unemployment.

## Deliverables
- `Phillips_test.ipynb`: clean, commented notebook following the steps above.
- 4–6 finished figures.
- Regression summary table by era.
- A short write-up (about 1 page) answering the question, covering:
  - whether a stable tradeoff exists (expected answer: not over the full sample),
  - how the 1970s support the Friedman–Phelps expectations critique,
  - the flat curve of 2008–2019 and what 2021–2023 suggests,
  - limitations: headline CPI, no expectations data, OLS with autocorrelated
    time series.

## Checklist
- [ ] Load and merge data; verify dates
- [ ] Inflation, change in inflation, and lag variables
- [ ] Era column
- [ ] Summary statistics, correlations, rolling correlation
- [ ] Figures 1–5
- [ ] Regressions and results table
- [ ] Robustness check(s)
- [ ] Write-up
- [ ] Export notebook to HTML
