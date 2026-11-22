## Exercise 1 — Failing feature test

```text
$ python -m pytest tests/test_leakage.py -k roll_3 -q --tb=short
F                                                               [100%]
============================== FAILURES ===============================
_____________ test_roll_3_is_mean_of_previous_three_days ______________
.venv/lib/python3.14/site-packages/pandas/core/indexes/base.py:3641: in get_loc
    return self._engine.get_loc(casted_key)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
pandas/_libs/index.pyx:168: in pandas._libs.index.IndexEngine.get_loc
    ???
pandas/_libs/index.pyx:197: in pandas._libs.index.IndexEngine.get_loc
    ???
pandas/_libs/hashtable_class_helper.pxi:7668: in pandas._libs.hashtable.PyObjectHashTable.get_item
    ???
pandas/_libs/hashtable_class_helper.pxi:7676: in pandas._libs.hashtable.PyObjectHashTable.get_item
    ???
E   KeyError: 'cnt_roll_3'

The above exception was the direct cause of the following exception:
tests/test_leakage.py:68: in test_roll_3_is_mean_of_previous_three_days
    assert by_date.loc[date, "cnt_roll_3"] == pytest.approx(expected)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.venv/lib/python3.14/site-packages/pandas/core/indexing.py:1199: in __getitem__
    return self.obj._get_value(*key, takeable=self._takeable)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.venv/lib/python3.14/site-packages/pandas/core/frame.py:4489: in _get_value
    series = self._get_item(col)
             ^^^^^^^^^^^^^^^^^^^
.venv/lib/python3.14/site-packages/pandas/core/frame.py:4969: in _get_item
    loc = self.columns.get_loc(item)
          ^^^^^^^^^^^^^^^^^^^^^^^^^^
.venv/lib/python3.14/site-packages/pandas/core/indexes/base.py:3648: in get_loc
    raise KeyError(key) from err
E   KeyError: 'cnt_roll_3'
======================= short test summary info =======================
FAILED tests/test_leakage.py::test_roll_3_is_mean_of_previous_three_days - KeyError: 'cnt_roll_3'
1 failed, 7 deselected in 1.15s
```

## Fault 1 — Forbidden column in `feature_columns`

### Exact change

```diff
--- /tmp/seed-fault-features.py.orig    2026-11-21 19:22:32
+++ src/features.py     2026-11-21 19:22:32
@@ -82,7 +82,7 @@
 def feature_columns(df: pd.DataFrame) -> list[str]:
     """Numeric columns that are safe to feed the model."""
     numeric = df.select_dtypes(include="number").columns
-    return [c for c in numeric if c not in FORBIDDEN]
+    return [c for c in numeric if c not in FORBIDDEN] + ["casual"]
```

### PostToolUse hook stderr

```text
Leakage tests failed after editing /Users/ganesh/Downloads/bikeshare-open-starter/src/features.py:
feats =         dteday  instant  season  ...  cnt_lag_7   cnt_roll_7   cnt_roll_3
0   2011-01-10     10.0  WINTER  ...     134....142857  2183.333333
713 2012-12-31    731.0  WINTER  ...      920.0  1531.428571  2077.333333

[714 rows x 29 columns]

    def test_no_forbidden_features(feats):
        """Guards the gate, not the data: feature_columns() excludes FORBIDDEN by
        construction, so this cannot discover a *new* leaky column. What it
        catches is someone editing feature_columns() or FORBIDDEN -- which is
        exactly the fault the Step 7 hook demonstrates."""
        cols = set(feature_columns(feats))
>       assert not (cols & FORBIDDEN), f"forbidden columns in feature set: {cols & FORBIDDEN}"
E       AssertionError: forbidden columns in feature set: {'casual'}
E       assert not ({'atemp', 'casual', 'cnt_lag_1', 'cnt_lag_7', 'cnt_roll_3', 'cnt_roll_7', ...} & {'casual', 'cnt', 'cnt_2d_bfr', 'days_since_2011', 'instant', 'registered', ...})

tests/test_leakage.py:31: AssertionError
======================= short test summary info =======================
FAILED tests/test_leakage.py::test_no_forbidden_features - Assertion...
1 failed, 7 passed in 0.84s
```

### Failing test output

```text
$ python -m pytest tests/test_leakage.py -k test_no_forbidden_features -q --tb=short
F                                                                        [100%]
=================================== FAILURES ===================================
__________________________ test_no_forbidden_features __________________________
tests/test_leakage.py:31: in test_no_forbidden_features
    assert not (cols & FORBIDDEN), f"forbidden columns in feature set: {cols & FORBIDDEN}"
E   AssertionError: forbidden columns in feature set: {'casual'}
E   assert not ({'atemp', 'casual', 'cnt_lag_1', 'cnt_lag_7', 'cnt_roll_3', 'cnt_roll_7', ...} & {'casual', 'cnt', 'cnt_2d_bfr', 'days_since_2011', 'instant', 'registered', ...})
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_no_forbidden_features - AssertionError: fo...
1 failed, 7 deselected in 0.78s
```

### Repair and turn count

Claude removed `+ ["casual"]` from `feature_columns()` and ran `make test`. The repair took **1 turn** after I requested it, with one edit and one verification command.

```text
................                                                         [100%]
16 passed in 1.66s
```

## Fault 2 — Removed the shift from `cnt_roll_7`

### Exact change

```diff
diff --git a/src/features.py b/src/features.py
index 76e492d..a6385db 100644
--- a/src/features.py
+++ b/src/features.py
@@ -73,7 +73,7 @@ def build_features(df: pd.DataFrame) -> pd.DataFrame:
     # If you add a longer lag, raise LAG_WARMUP in src/config.py.
     out["cnt_lag_1"] = out[TARGET].shift(1)
     out["cnt_lag_7"] = out[TARGET].shift(LAG_WARMUP)
-    out["cnt_roll_7"] = out[TARGET].shift(1).rolling(LAG_WARMUP).mean()
+    out["cnt_roll_7"] = out[TARGET].rolling(LAG_WARMUP).mean()
     out["cnt_roll_3"] = out[TARGET].shift(1).rolling(ROLL_SHORT).mean()
 
     return out.dropna().reset_index(drop=True)
```

### PostToolUse hook stderr (excerpt)

Claude Code clipped the displayed hook output. These lines were visible in the original hook result.

```text
E             comparison failed
E             Obtaine
E             Expected: 4414.857142857143 ± 0.00441486

tests/test_leakage.py:57: AssertionError
============================================
FAILED tests/test_lea_excludes_today -ass...
1 failed, 7 passed in
```

### Failing test output

```text
$ python -m pytest tests/test_leakage.py -k test_rolling_mean_excludes_today -q --tb=short
F                                                                        [100%]
=================================== FAILURES ===================================
_______________________ test_rolling_mean_excludes_today _______________________
tests/test_leakage.py:57: in test_rolling_mean_excludes_today
    assert feats.set_index("dteday").cnt_roll_7[date] == pytest.approx(expected)
E   assert np.float64(4389.0) == 4414.857142857143 ± 0.00441486
E     
E     comparison failed
E     Obtained: 4389.0
E     Expected: 4414.857142857143 ± 0.00441486
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_rolling_mean_excludes_today - assert np.fl...
1 failed, 7 deselected in 0.78s
```

### Repair and turn count

Claude restored `.shift(1)` before `.rolling(LAG_WARMUP)` in `cnt_roll_7` and ran `make test`. The repair took **1 turn** after I requested it, with one edit and one verification command.

```text
................                                                         [100%]
16 passed in 1.64s
```