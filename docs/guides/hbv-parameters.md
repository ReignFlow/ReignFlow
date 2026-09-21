# Use exported HBV parameters

A successful dHBV test exports the parameters generated from the weights used
for that test. The default files are:

```text
results/hbv_parameters.nc
results/hbv_parameters.metadata.json
```

These are generated physical-model parameters, not a neural checkpoint and
not the time-varying HBV states. Disable them with
`--export_hbv_parameters False`.

## NetCDF fields

| Variable | Meaning |
|---|---|
| `hbv_parameter_normalized` | Sigmoid parameter fractions per basin/component/name |
| `hbv_parameter_physical` | Parameters mapped into the model's physical ranges |
| `hbv_parameter_lower_bound`, `hbv_parameter_upper_bound` | Mapping bounds |
| `component_weight` | Weights for combining component discharges |
| `routing_parameter_normalized`, `routing_parameter_physical` | Routing fractions and mapped parameters |
| `routing_parameter_lower_bound`, `routing_parameter_upper_bound` | Routing mapping bounds |
| `routing_gamma_parameter_effective` | Gamma shape/scale after positivity offsets |
| `routing_uh_weight` | Discrete 15-day routing unit hydrograph |

Dimensions include `station`, `component`, `parameter`, `routing_component`,
`routing_parameter`, and `lag`. The standard model has 12 HBV parameter names,
two routing parameters, and lag midpoints 0.5 through 14.5 days. Station IDs
are strings in the selected run order. The [small tutorial](../tutorials/dhbv.md)
produces three stations and two components.

```python
import xarray as xr

with xr.open_dataset("output/your-run-directory/results/hbv_parameters.nc") as ds:
    print(dict(ds.sizes))
    print(ds["hbv_parameter_physical"])
    print(ds["routing_uh_weight"].sum("lag"))
```

## Reuse the ensemble correctly

The standard model uses an arithmetic mean of component **discharges**.
Averaging component parameters and running one HBV model generally produces
a different result. For software that accepts one component, train with
`--nmul 1` or simulate all components separately and combine their discharge.

Parameter reuse also requires matching HBV equations, parameter meanings,
forcing units, initial-state/warm-up treatment, and routing discretization.
The export alone does not guarantee that another HBV implementation reproduces
ReignFlow's hydrograph. Snowpack, soil moisture, and storage trajectories are
not included.

## Metadata

The JSON sidecar binds the file to the selected checkpoint/bundle, config/data
hashes, ordered stations, parameterization and test periods, physical ranges,
component-combination rule, and routing convention. It uses portable identifiers
instead of embedding a full local configuration. Retain it alongside the NetCDF
and inspect both before sharing.
