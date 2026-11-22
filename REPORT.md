# Bikeshare Demand Forecasting: Part 5 Report

Daily Capital Bikeshare counts, 2011-2012. Target: `cnt` (total daily rides).
Split by date: train = 2011 (348 rows), validation = H1 2012 (182 rows),
test = H2 2012 (184 rows).

## Results

"Before" is `~/bikeshare-baseline-metrics.json` (14 features, no `cnt_roll_3`).
"After" is `reports/metrics.json` (15 features, with `cnt_roll_3`). The mean
and persistence baselines do not depend on the feature set, so their rows are
identical before and after.

| Model | Pipeline | Valid R² | Valid MAE | Test R² | Test MAE |
|---|---|---|---|---|---|
| Mean baseline | before | -0.9318 | 2002.2 | -2.1441 | 2847.7 |
| Mean baseline | after | -0.9318 | 2002.2 | -2.1441 | 2847.7 |
| Persistence | before | 0.4507 | 900.6 | 0.5099 | 840.1 |
| Persistence | after | 0.4507 | 900.6 | 0.5099 | 840.1 |
| Ridge | before | 0.6475 | 832.9 | 0.5833 | 947.1 |
| Ridge | after | 0.6439 | 836.5 | 0.58 | 946.2 |
| HGB | before | 0.2185 | 1296.6 | -0.4975 | 1972.6 |
| HGB | after | 0.1983 | 1311.6 | -0.5338 | 1998.7 |

MAE is in rides per day. The starter pipeline (`python -m src.train`) prints
test scores along with validation scores, so baseline test numbers were
displayed during setup, before any of my modeling work. Model and feature
decisions (adding `cnt_roll_3`, keeping it, the Exercise 4 experiment) were
based on validation scores only; the test columns are reported here for the
final comparison. Negative R² values are findings, not bugs: the mean
baseline is fixed at the 2011 average, and 2012 ridership is far higher.

## Headline finding

A model given `casual` and `registered` scores an R² of 1.000. That figure is
the assignment's demonstration result (recorded in the starter's
`docs/DECISIONS.md` and README, and reproduced by the starter's
`src/leakage_demo.py`); I have no evidence that I ran the demo myself, and it
is not in either metrics file. The score is invalid: `casual + registered == cnt` exactly on every row, so the
model has learned addition of two components of the very quantity being
predicted, neither of which is known at prediction time. Once those columns
are excluded, the honest models are much less impressive. Ridge is the best
valid model on validation (R² 0.6439, MAE 836.5 in the final pipeline) and
beats persistence (0.4507, 900.6) and the mean baseline (-0.9318, 2002.2). On
test its lead mostly disappears: ridge R² is 0.58 against persistence 0.5099,
and persistence has the lower MAE (840.1 vs 946.2). HGB is worse than
persistence on both splits, and its negative test R² (-0.5338) reflects
that trees trained on 2011 cannot extrapolate to 2012, when ridership was
roughly 65% higher. It still beats the mean baseline (-2.1441) on test.

`cnt_roll_3` worsened both models on validation (ridge R² 0.6475 to 0.6439,
HGB R² 0.2185 to 0.1983), and it was kept in the submitted pipeline to
reproduce the comparison.

## Exercise 4: target transformation

**Method.** Instead of predicting `cnt` directly, HGB was trained on the
target `cnt - cnt_lag_1` (today's count minus yesterday's count). At scoring
time, `cnt_lag_1` was added back to each prediction, so R² and MAE are
computed on the original ride-count scale. The features, dates, and HGB
settings (`random_state=0`, `max_iter=300`) were identical to the direct
model, which used the same 15 features, and the split came from
`temporal_split`.

| Model | Valid R² | Valid MAE |
|---|---|---|
| Persistence (yesterday's count) | 0.4507 | 900.6 |
| HGB, direct on `cnt` | 0.1983 | 1311.6 |
| HGB, delta target | 0.6251 | 805.4 |

**Why it is defensible.** For one-day-ahead forecasting, yesterday's observed
count is available at prediction time, so using it as an anchor is legitimate.
The trees then only model the day-to-day change, which is far less dependent
on the overall ridership level than the count itself. That removes the main
failure of direct HGB, which cannot predict above the 2011 range.

**Why it stayed an experiment.** It is one run, with one seed, on 182
validation days, so the size of the gap is indicative and not precise. It was
also not part of the pipeline that produced the submitted metrics, which keeps
the untransformed target.

**Evaluation scope.** The transformed model was evaluated only on validation;
its test split was never scored or inspected. The final pipeline's test
scores in Section 1 were used only after the modeling decisions were fixed.

## Exercise 5: ridge vs persistence

If the deployment goal is lower average absolute error, persistence is the
better choice. On the final test numbers, persistence has an MAE of 840.1
rides against ridge's 946.2, so it is wrong by about 106 rides per day less
on average. Ridge's advantage is in R² (0.58 vs 0.5099, a gap of about
0.07), which weights large errors more heavily and so rewards ridge for
doing better on the biggest misses, but those do not drive average absolute
error. So the tradeoff is roughly 106 rides of average daily error against
0.07 of R². Persistence also needs no fitting, no features, and no retraining
as ridership drifts, while ridge needs the full feature pipeline for a worse
MAE. If the cost of a large miss mattered more than the average miss, ridge
would be the defensible choice.

## Attribution

I tried local Ollama during setup and used Claude Sonnet 5.5
(`claude-sonnet-5-5`) for the assignment exercises. The starter ships Ollama
files (`model/Modelfile`, `model/check.sh`, `agent/audit.py`), but nothing in
the repo records my trying it, so that part rests on my own statement. The
Claude model's parameter count was unavailable.

**What the agent generated** (session evidence and commit messages)
- The `cnt_roll_3` feature and its test (commits `349b576`, `e6e9325`).
- The validation-only HGB delta-target experiment, as a temporary script
  outside the repo (Exercise 4); it never touched the test split.
- The leakage-auditor subagent run on the teammate's repo, saved verbatim in
  `docs/AUDIT.md` (Exercise 3). It found no problems at commit `517be0d`.
- The `CLAUDE.md` and `.gitignore` updates, the `docs/DECISIONS.md` wording,
  and the drafts of this report.

**What I decided** (`docs/DECISIONS.md`)
- Keeping `cnt_roll_3` in the submitted pipeline to reproduce the comparison,
  while judging it worse for deployment (2026-11-21).
- Keeping the delta target as a validation-only experiment and the
  untransformed pipeline for the submitted metrics (2026-11-22).
- The CLAUDE.md and `.gitignore` changes for Part 5 (2026-11-22), which target
  `reports/` being committed.
- Which teammate repo to audit, the verdict on that audit (left for me to
  write), and when to commit.

**Inherited from the starter** (commit `cb95478`, not my choices)
- The forbidden-column list and reasons, the date split (train 2011,
  validation H1 2012, test H2 2012), both baselines, the ridge and HGB
  configurations, the lag features, the original `CLAUDE.md`, and the first
  five `docs/DECISIONS.md` lines.
- The `leakage-auditor` agent, the `check_leakage.sh` hook, the `new-feature`
  skill, and `agent/audit.py`. The `seed-fault` skill first appears in commit
  `b3eb822`, so I treat it as added during the exercises, not inherited.

**Could not be verified**
- Ollama use and the switch to Sonnet 5.5: no repo evidence.
- Which commits the agent wrote versus I wrote: every commit is authored as
  `ganeshkumar321` with no co-author trailer, so git history cannot separate
  them.
- The Exercise 2 fault-seeding steps: I did not re-read
  `docs/SEEDED_FAULTS.md` for this report.
