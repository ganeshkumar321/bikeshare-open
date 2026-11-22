# Decisions

One line per methodological decision: what you chose, why, and what it changed.
Agents append here via the `/new-feature` skill; you append here by hand.

- Exclude `casual`, `registered`: they sum to `cnt` exactly. Leaky model R^2 1.000 -> honest ridge 0.583.
- Exclude `cnt_2d_bfr`: a shipped 2-day lag we cannot test the alignment of. Build our own lags instead.
- Exclude `instant`, `yr`, `days_since_2011`: time indexes let a tree predict a flat constant past 2011 and a linear model extrapolate a trend we have no grounds for.
- Split 2011 / H1 2012 / H2 2012 by date. Test set touched once, after all feature work.
- Report mean and persistence baselines with every score. Persistence (test R^2 0.510, MAE 840) is within 0.07 R^2 of ridge and beats it on MAE; that caveat goes in the report.

- 2026-11-21: Added `cnt_roll_3` using ride counts from the three completed calendar days before prediction. On validation, ridge R² fell from 0.6475 to 0.6439 (−0.0036) and MAE rose from 832.9 to 836.5 (+3.6 rides). HGB R² fell from 0.2185 to 0.1983 (−0.0202) and MAE rose from 1296.6 to 1311.6 (+15.0 rides). I kept the feature in the submitted pipeline to reproduce the comparison and would exclude it from deployment because both models performed worse.

- 2026-11-22: For Exercise 4, I trained HGB to predict the change from yesterday's ride count and added yesterday's count back before scoring. On validation, this improved HGB R² from 0.1983 to 0.6251 and reduced MAE from 1311.6 to 805.4 rides. It also beat persistence (R² 0.4507, MAE 900.6). The transformation is defensible for one-day-ahead forecasting because yesterday's observed count is available and anchors the forecast to recent demand. I kept it as a validation experiment and retained the untransformed pipeline for the submitted metrics. I did not score the test split for this experiment.

- 2026-11-22: Updated CLAUDE.md for the Part 5 submission: added the `cnt_roll_3` rule (three prior calendar days, reindex first, shift before rolling) and a pointer to the `/seed-fault` skill, keeping the file under 60 lines with the forbidden columns, temporal split, both baselines, and `make all` intact. Also stopped ignoring `reports/` and anchored `audit.jsonl` to `/audit.jsonl` in .gitignore so `reports/metrics.json` and `reports/audit.jsonl` can be committed.
