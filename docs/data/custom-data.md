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

The [custom-data example](../tutorials/custom-data.md) creates a small NetCDF,
runs a complete LSTM training command, and checks the saved output layout.
Replace its arrays with your own daily measurements once the schema is clear.

`CAMELS` supplies a profile; explicit options replace its station list,
variables, attributes, and dates. Change all of these to match your file.
Use `--station_file` instead of `--all_stations` for a fixed subset, and keep
ID strings, including leading zeros, unchanged.

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
