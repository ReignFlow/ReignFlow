# Variables, periods, and normalization

ReignFlow merges the selected dataset profile with CLI overrides. A variable
list replaces the whole profile list. The order determines model feature
order and must remain consistent for checkpoint evaluation.

## Variables

```text
--time_series_variables daymet_prcp,daymet_tmean,daymet_pet
--target_variables Runoff
--static_variables area_gages2,elev_mean
```

Use `--static_variables None` to disable attributes. `--add_coords` appends
`lat` and `lon` as two more static channels. The effective input dimension is
forcing count plus static count, including those coordinates; output width is
the target count. These dimensions are derived rather than supplied as CLI
options.

## Periods and history

Each date argument is an inclusive `start,end` pair. Choose disjoint train,
validation, and test periods if validation influences model selection. The
code does not automatically enforce this scientific separation.

| Task/profile | Sample length | History prepended before a requested split |
|---|---|---|
| Neural regression | `seq_len` | `seq_len - pred_len` days |
| Neural forecast | `seq_len + pred_len` | `seq_len` days |
| dHBV regression profile | `seq_len` | None; warm-up is internal |

Each sample is one basin's input window. For a neural model with
`seq_len=4` and `pred_len=1`:

| Input days | Regression target | Forecast target |
|---|---|---|
| 1–4 | Day 4 | Day 5 |
| 2–5 | Day 5 | Day 6 |
| 3–6 | Day 6 | Day 7 |

Here, regression estimates runoff on the last input day; forecasting predicts the
following day. Training batches combine windows across basins according to
the [sampling policy](training-control.md#window-sampling).

For neural models, earlier history supplies inputs; targets begin in the
requested period when complete history is available. Training statistics are
fitted over the **loaded training view**, including this history. They are
not fitted separately for validation or test.

If history is truncated, training/validation may omit the first target days
and print a warning. Ordinary testing additionally requires the reconstructed
length to match the full nominal daily test interval; missing test history can
therefore cause a failure. Include enough preceding dates or move the test start.

dHBV's test begins exactly one day after training ends. Its training and test
splits must each contain at least `seq_len` days; enabled validation must also
fit that window. See [the dHBV tutorial](../tutorials/dhbv.md).

## Normalization

All splits use the training scaler and the inverse formula
`z * (std + 1e-5) + mean`. The [data reference](../data/camels.md#normalization-contract)
defines the fitting pool, standard-deviation conventions, and missing values.
dHBV's physical outputs bypass inverse normalization.

### Gamma forcing transform

`--gamma_norm_variables daymet_prcp` applies `log10(sqrt(x) + 0.1)` to the
selected forcing before z-score fitting. Multiple names are comma-separated;
`none` explicitly disables the transform; omission inherits the profile.
Names must be selected forcing variables. Duplicates and finite negative
values are rejected; zero is valid and NaNs remain missing.

This transform changes normalized forcing only. dHBV's raw P/T/PET remains
physical. The transform selection is part of the normalization contract, so
changing it can invalidate checkpoint/bundle replay.

### Missing values

Use NetCDF missing-value metadata when available. For a legacy file, an
explicit raw-value rule can be passed as:

```text
--missing_values '{"*":[-999,-9999],"legacy_flow":[-12345]}'
```

Per-variable rules replace the `*` fallback; values are interpreted before
CF scale/offset decoding. Undeclared suspicious values trigger a warning and
remain unchanged. A rule with no matches is allowed.

## What to retain for evaluation

Keep station order, variables, periods, model/window settings, missing-value
rules, and normalization settings. Keep the effective `--do_eval` state as
well: a loaded validation split enters the decoded-data fingerprint. The
run's `configs.sh` records resolved settings; see
[checkpoint evaluation](evaluate-and-share.md).
