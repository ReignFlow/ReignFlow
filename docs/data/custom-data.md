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

The current loader uses calendar days for history and output-length checks;
the Dataset requests daily calendar features. Hourly or irregular data require
code changes and validation, not just a renamed variable or a `freq` value.
The reader does not diagnose every possible irregular time axis up front.

## Override the inherited defaults

For example, a file with `rain`, `temperature`, `flow`, and no static attributes
needs these **argument fragments** added to a complete training command:

```text
--data CAMELS
--input_nc_file path/to/daily.nc
--all_stations
--time_series_variables rain,temperature
--target_variables flow
--static_variables None
--train_date_list 2000-01-01,2004-12-31
--val_date_list 2005-01-01,2005-12-31
--test_date_list 2006-01-01,2006-12-31
```

Without the overrides, the profile still requests its built-in basin IDs,
forcing names, attributes, and dates. `--station_file` can replace
`--all_stations` for a fixed subset.

The full source checkout includes `examples/quickstart/create_demo_data.py`,
a small working schema example described in the
[quick start](../getting-started/quickstart.md). That script will accompany the
code release. Its runoff equation is an artificial fixture, not a hydrological
dataset preparation method.

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
