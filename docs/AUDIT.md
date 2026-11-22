# Leakage audit of teammate repo (Exercise 3)

- Repo: https://github.com/adityakakarla/DSC180A-Methodology-6
- Audited commit: `517be0d9dfefd317c994e5f9909c027ce990b138`
- Local checkout: `../DSC180A-Methodology-6` (source files untouched)
- Auditor: `leakage-auditor` subagent (`.claude/agents/leakage-auditor.md`)

## Subagent output (verbatim)

All five checks pass for commit 517be0d in /Users/ganesh/Downloads/DSC180A-Methodology-6, so I have no findings to report. I ran the code in memory with `-I` and `PYTHONDONTWRITEBYTECODE=1`, and nothing was written to the checkout.

- **Target leakage:** The 15 feature columns exclude `casual`, `registered`, `cnt` and `cnt_2d_bfr`, and no feature adds or subtracts them. The lag, rolling mean and rolling std features are all built from `cnt.shift(1)` or `cnt.shift(7)`. The highest correlation with `cnt` is 0.85, for `cnt_roll_7`.
- **Temporal ordering:** `shift(1)` comes before every rolling aggregate, and the split is by date. The scaler sits inside the pipeline and is fitted on train only, and the persistence baseline is not fitted at all.
- **Calendar gaps:** `build_features` calls `reindex_to_calendar` before any lag, so 2011-03-10 becomes an explicit NaN row. The tests check the lags against calendar dates.
- **Silent row loss:** The raw file has no NaNs. 728 rows become 714, a loss of 14. The bound is 7 warm-up rows plus 7 for the one missing day, which is also 14, so the loss is exactly the expected amount. `encode_weather` raises on unknown labels. The bound is only asserted in `tests/test_contract.py`, not inside the pipeline itself.
- **Unreported baseline:** `baseline_mean` and `baseline_persistence` are scored alongside ridge and hgb, for both the full and the `before_feature` runs. I recomputed the scores and they match `reports/metrics.json`. On test, persistence has R² 0.51 and MAE 840.1, and ridge has R² 0.64 and MAE 840.3. `REPORT.md` states this MAE tie.

## My verdict

_(to be written by me)_
