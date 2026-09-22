# Train with custom daily data

Create a NetCDF with your own variable names and train an LSTM without adding
a new dataset profile. This example uses two synthetic stations to demonstrate
the [required data structure](../data/custom-data.md#required-structure).

## Before you start

Complete [installation](../getting-started/installation.md) and run from the full
source checkout with the environment active. All commands below use CPU execution.

## 1. Create a NetCDF

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
for the first training windows; this example has no static attributes.
The artificial flow equation is only a formatting fixture.

## 2. Train and evaluate

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

The explicit options replace the CAMELS profile's station list, variables,
attributes, and dates. Validation selects the saved weights used for testing.

## 3. Inspect the result

Follow the printed run directory under `output/custom/lstm/`. Expect physical
prediction and observation arrays of `[2, 61, 1]`, representing two stations,
61 test days, and one target. The target unit is mm/day and the test period
is 1 May–30 June 2000.

Open `results/feature_flow.png` to inspect the first station, or load
`results/pred.npy` and `results/true.npy`. Use the
[quick-start interpretation](../getting-started/quickstart.md#3-inspect-the-predictions)
to read NSE and RMSE.

## Next steps

Replace the artificial arrays with daily measurements, checking
[units and missing observations](../data/custom-data.md#units-and-missing-observations).
Then select your [stations](../guides/station-selection.md),
[variables and periods](../guides/data-selection.md), and
[model](../models/overview.md).
