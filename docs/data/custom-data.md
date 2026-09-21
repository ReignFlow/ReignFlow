# Use your own daily data

You can supply another NetCDF with `--input_nc_file` and override the CAMELS
profile's variables, dates, and stations. No new profile is needed when these
options express your dataset.

## Required structure

- Use dimensions and coordinates named `station_ids` and `time`.
- Keep station IDs unique strings, including any leading zeros.
- Provide a sorted, unique, continuous daily time axis. Keep missing daily
  observations as NaN on that axis instead of deleting dates.
- Store each forcing and target on `[station_ids, time]` and each numeric
  attribute on `[station_ids]`.
- Supply `lat` and `lon` only if you enable `--add_coords`.

The reader stacks the configured forcing and target variables into
`[basin, time, feature]` arrays internally. Each NetCDF variable uses the
dimensions listed above.

The current loader uses calendar days for history and output-length checks;
the Dataset requests daily calendar features. Hourly or irregular data require
code changes and validation, not just a renamed variable or a `freq` value.
The reader does not diagnose every possible irregular time axis up front.

## A minimal file and complete run

Use this synthetic schema example to check the interface, then replace the
arrays with your own daily measurements. The artificial flow equation is
only a formatting fixture.

<!-- example: custom-data -->
```bash
python - <<'PY'
from pathlib import Path
import numpy as np
import pandas as pd
import xarray as xr

dates = pd.date_range("1999-12-01", "2000-06-30", freq="D")
rng = np.random.default_rng(42)
rain = rng.gamma(2.0, 1.0, size=(2, len(dates)))
temperature = 10.0 + rng.normal(size=rain.shape)
flow = 0.35 * rain + 0.03 * temperature
dims = ("station_ids", "time")
ds = xr.Dataset(
    {"rain": (dims, rain, {"units": "mm day-1"}),
     "temperature": (dims, temperature, {"units": "degC"}),
     "flow": (dims, flow, {"units": "mm day-1"})},
    coords={"station_ids": ["001", "002"], "time": dates},
)
Path("output/custom").mkdir(parents=True, exist_ok=True)
ds.to_netcdf("output/custom/daily.nc")
print(dict(ds.sizes))
ds.close()
PY
```

The file has two stations and 213 daily timestamps. December supplies history
for the first training windows. There are no static attributes in this example.

<!-- example: custom-train -->
```bash
python -m reignflow --task_name regression --model LSTM --data CAMELS \
  --input_nc_file output/custom/daily.nc --all_stations \
  --time_series_variables rain,temperature --target_variables flow \
  --static_variables None \
  --train_date_list 2000-01-01,2000-03-31 \
  --val_date_list 2000-04-01,2000-04-30 \
  --test_date_list 2000-05-01,2000-06-30 \
  --seq_len 14 --pred_len 1 --d_model 16 --dropout 0 \
  --batch_size 32 --epochs 2 --learning_rate 0.001 \
  --save_best --device cpu --seed 42 --output_dir output/custom/lstm
```

Expect physical prediction/observation arrays of `[2, 61, 1]` and
`results/feature_flow.png` in the printed run directory. Use the
[quick-start interpretation](../getting-started/quickstart.md#3-inspect-the-predictions)
to read the result.

`CAMELS` supplies a profile here; the explicit options replace its station
list, variables, attributes, and dates. With real data, change all of these
to match your file. Use `--station_file` instead of `--all_stations` for a
fixed subset. Keep ID strings, including leading zeros, unchanged.

## Units and missing observations

For the documented hydrological workflow, use runoff depth in mm/day.
Convert discharge using basin area before loading, and retain useful NetCDF
unit and missing-value metadata. Ordinary neural profiles do not generally
validate or convert target units; reported RMSE follows the supplied target
units and `BasinNormalizedMSE` weights depend on them.

Input NaNs are filled with zero after normalization, equivalent to the fitted
mean in normalized space. Target NaNs remain masked. This is not a missingness
indicator or a learned imputation model.

At least one selected station must contain forcing and target observations.
Even standalone `--do_test` loads training and test data and validates their
fingerprints. The current CLI is an evaluation workflow with observations;
it is not an observation-free prediction service.

## dHBV requires physical roles

For dHBV, begin with `CAMELS_dHBV` and provide precipitation, temperature, PET,
and runoff with the profile's names and units. Overriding only the forcing
names does not rename the physical-role mapping. A different mapping requires
a profile change in `config_dataset_hbv.py`, appropriate tests, and a new
compatibility record.

Maintain complete physical forcing where possible. Missing-value handling in
the normalized parameterizer is distinct from the physical HBV branch; see
[dHBV](../models/dhbv.md#physical-forcing).
