## Exercise 1 — Failing feature test

```text
$ python -m pytest tests/test_leakage.py -k roll_3 -q --tb=short
F                                                                        [100%]
=================================== FAILURES ===================================
__________________ test_roll_3_is_mean_of_previous_three_days __________________
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
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_roll_3_is_mean_of_previous_three_days - Ke...
1 failed, 7 deselected in 1.05s
```
