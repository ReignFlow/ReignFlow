# CAMELS NetCDF

CAMELS (Catchment Attributes and Meteorology for Large-sample Studies) combines
meteorological inputs, observed streamflow, and catchment attributes for US
basins. Cite [Newman et al. (2015)](https://doi.org/10.5194/hess-19-209-2015)
for the hydrometeorological data and
[Addor et al. (2017)](https://doi.org/10.5194/hess-21-5293-2017) for the attributes.

ReignFlow loads daily forcing, runoff, and basin attributes from one NetCDF
file. Downloaded archives and generated NetCDF files stay outside version
control. The [quick start](../getting-started/quickstart.md) supplies synthetic
data if you want to check the installation first.

## Build the file

Run from the repository root:

```bash
python examples/data_preparation/01.download_camels.py \
  --output-dir examples/data_preparation/data/CAMELS_raw
python examples/data_preparation/02.prepare_camels.py \
  --input-dir examples/data_preparation/data/CAMELS_raw \
  --output-dir examples/data_preparation/data/CAMELS_processed \
  --start-date 1980-01-01 --end-date 2014-12-31
```

The downloader retrieves attributes and forcing/streamflow archives; use
`--no-extended` to skip extended Maurer and NLDAS products. Full downloads and
preparation can take substantial time and storage. The preparation script
merges products, calculates mean temperature and Hargreaves PET, converts
streamflow to runoff depth, and writes `CAMELS.nc`.

| Preparation option | Meaning |
|---|---|
| `--input-dir` | Downloaded input directories |
| `--output-dir` | Destination for `CAMELS.nc` |
| `--start-date`, `--end-date` | Inclusive daily time axis |
| `--basin-list` | Optional text file containing basin IDs |

Both scripts provide `--help`. These commands have CLI coverage tests; the
documentation smoke suite does not redownload CAMELS or reproduce a full build.

## Schema

| Field | Dimensions | Meaning |
|---|---|---|
| `station_ids` | `station_ids` | Unique string basin IDs |
| `time` | `time` | Daily timestamps |
| `lat`, `lon` | `station_ids` | Coordinates, used when `--add_coords` is enabled |
| `daymet_*`, `nldas_*`, `maurer_*` | `station_ids, time` | Meteorological variables |
| `QObs` | `station_ids, time` | Observed discharge in `ft3 s-1` |
| `Runoff` | `station_ids, time` | Area-normalized runoff in `mm day-1` |
| Numeric basin attributes | `station_ids` | Climate, terrain, vegetation, soil, and geology |

Forcing names include `prcp`, `srad`, `tmax`, `tmin`, `dayl`, and `vp`, with
the product prefix. Derived variables include `pet` and `tmean`. The commonly
used full record spans 1980–2014 and 671 basins; subsets have different sizes.
Inspect your file rather than assume all products or stations are present.

```python
import xarray as xr

with xr.open_dataset("examples/data_preparation/data/CAMELS_processed/CAMELS.nc") as ds:
    print(dict(ds.sizes))
    print(ds["Runoff"].attrs)
    print(list(ds.data_vars))
```

## Dataset profiles

`--data` supplies defaults. CLI selections replace the corresponding profile
values; they do not add to a variable list. Both profiles resolve their default
file relative to the source checkout:
`examples/data_preparation/data/CAMELS_processed/CAMELS.nc`.
Use `--input_nc_file path/to/CAMELS.nc` for a different location.

The following counts, variables, and dates are generated from the actual
profile factories. CAMELS is suitable for LSTM and Transformer;
CAMELS_dHBV supplies the physical roles needed by dHBV.

<!-- BEGIN GENERATED: profiles -->

| Profile | Stations | Forcings | Static attributes | Target |
|---|---:|---:|---:|---|
| `CAMELS` | 531 | 6 | 26 | `Runoff` |
| `CAMELS_dHBV` | 671 | 3 | 32 | `Runoff` |

### `CAMELS` defaults

- Train: `1980-10-01` through `1995-09-30` (inclusive).
- Val: `2010-10-01` through `2014-09-30` (inclusive).
- Test: `1995-10-01` through `2010-09-30` (inclusive).

Forcings: `daymet_prcp`, `daymet_srad`, `daymet_tmax`, `daymet_tmin`, `daymet_dayl`, `daymet_vp`.

??? info "Static attribute names"

    `elev_mean`, `slope_mean`, `area_gages2`, `frac_forest`, `lai_max`, `lai_diff`, `gvf_max`, `gvf_diff`, `soil_depth_pelletier`, `soil_depth_statsgo`, `soil_porosity`, `soil_conductivity`, `max_water_content`, `sand_frac`, `silt_frac`, `clay_frac`, `carbonate_rocks_frac`, `geol_permeability`, `p_mean`, `pet_mean`, `aridity`, `frac_snow`, `high_prec_freq`, `high_prec_dur`, `low_prec_freq`, `low_prec_dur`.

### `CAMELS_dHBV` defaults

- Train: `1980-10-01` through `1995-09-30` (inclusive).
- Val: `2010-10-01` through `2014-09-30` (inclusive).
- Test: `1995-10-01` through `2010-09-30` (inclusive).

Forcings: `daymet_prcp`, `daymet_tmean`, `daymet_pet`.

??? info "Static attribute names"

    `p_mean`, `pet_mean`, `p_seasonality`, `frac_snow`, `aridity`, `high_prec_freq`, `high_prec_dur`, `low_prec_freq`, `low_prec_dur`, `elev_mean`, `slope_mean`, `area_gages2`, `frac_forest`, `lai_max`, `lai_diff`, `gvf_max`, `gvf_diff`, `dom_land_cover_frac`, `root_depth_50`, `soil_depth_pelletier`, `soil_depth_statsgo`, `soil_porosity`, `soil_conductivity`, `max_water_content`, `sand_frac`, `silt_frac`, `clay_frac`, `glim_1st_class_frac`, `glim_2nd_class_frac`, `carbonate_rocks_frac`, `geol_porostiy`, `geol_permeability`.

<!-- END GENERATED: profiles -->

## Normalization contract

Precipitation, temperature, and basin attributes have different units and
scales. Standardization rescales features to help optimization. Use training
statistics for exploratory normalization plots as well as model inputs.

The training loader fits one mean and population standard deviation per
forcing and target over the loaded stations and dates. Stations lacking all
forcing or all target observations are excluded from this fitting pool;
their positions are retained in the returned arrays. Static attributes use
the selected stations and sample standard deviation (`ddof=1`). A one-station
static standard deviation is therefore NaN and its normalized value becomes zero.

The fitted training view includes any prepended history for neural windows.
Validation and test reuse its statistics. The physical dHBV profile carries
warm-up inside the requested period and prepends no external history.

```python
epsilon = 1e-5
z = (q - mean) / (std + epsilon)
q = z * (std + epsilon) + mean
```

Missing normalized forcing and static values become zero; target NaNs remain
for masking. Zero standard deviations use the same epsilon. Floating-point
rounding still applies: the inverse is algebraically consistent, not a promise
of bitwise restoration after conversion to float32.

Earlier inverse normalization omitted epsilon. Its checkpoint/bundle contract
is incompatible with the corrected one; see [normalization compatibility](../guides/checkpoints.md#normalization-compatibility).
dHBV outputs physical values and bypasses this inverse.

## Missing values and units

NetCDF `_FillValue`/`missing_value` and CF scale/offset metadata are decoded.
For legacy sentinels without metadata, use a JSON argument such as
`--missing_values '{"*":[-999,-9999],"legacy_flow":[-12345]}'`.
Rules refer to raw stored values; a variable-specific rule replaces `*`.
Undeclared suspicious values produce warnings and are not silently guessed
to be missing. Conversion counts are recorded with the run.

dHBV validates the declared precipitation, temperature, PET, and target roles.
Its P/PET/Runoff units are `mm day-1`; temperature is `degC`, with recognized
equivalent spellings accepted. A missing unit declaration can use the explicit
profile unit with a warning; incompatible declared units fail. This validation
does not convert volumetric discharge to runoff depth.

## Data-boundary errors

Empty selections, absent variables/IDs, empty requested periods, infinite
values, and arrays without usable forcing/target coverage fail explicitly.
Supplying checkpoint statistics does not bypass these checks. Partial target
missingness is allowed; a usable window must still fit the loaded split.

See [variables and periods](../guides/data-selection.md),
[station selection](../guides/station-selection.md), and
[custom data](custom-data.md) before adapting another dataset.
