# LSTM

An LSTM carries hidden and cell states through the input window, allowing
earlier rainfall and catchment conditions to influence today's runoff estimate.

`--model LSTM` concatenates normalized forcing and repeated basin attributes,
runs a one-layer `torch.nn.LSTM`, then applies output dropout and a linear
projection to the target channels. The model returns the full input-length
sequence; the trainer uses its final `pred_len` steps.

The plain LSTM implementation is adapted from
[`kratzert/multiple_forcing`](https://github.com/kratzert/multiple_forcing).
See [acknowledgements and code provenance](../acknowledgements.md) for its
scope and the related loss implementations.

| Setting | Meaning |
|---|---|
| `--d_model` | Recurrent hidden size; default 256 |
| `--dropout` | Dropout after the recurrent sequence; default 0.1 |
| `--initial_forget_bias` | Value assigned to the recurrent forget-gate bias slice; default 3.0 |

The plain LSTM runs on CPU or CUDA. Begin with the [quick start](../getting-started/quickstart.md)
and use the [CAMELS tutorial](../tutorials/first-lstm.md) for observations.

## Losses and units

LSTM predicts normalized targets. `MaskedMSE` averages squared
errors over non-NaN observations. `BasinNormalizedMSE` additionally weights each
error using the basin's standard deviation from the **raw loaded training
targets**:

```text
loss = mean((z_prediction - z_observation)^2 / (basin_std + 0.1)^2)
```

The mean is over the valid target elements. `basin_std` uses NumPy's population
standard deviation; it is distinct from the global target standard deviation
used by the z-score transform.

Basin weighting downscales errors in basins with more variable runoff.
Actual contributions still depend on prediction errors, valid sample counts,
and the stabilizing `+0.1` term.

This is the formula implemented in this checkout. There is no `target_unit`
or `freq` CLI option, no required profile `target_unit`, and no timestep-aware
loss rescaling. Use daily, consistently scaled targets for the CAMELS workflow.
Changing target units changes the basin-weighted objective even when targets
are globally normalized.

## Evaluation

Window predictions and observations are reconstructed and inverse-normalized
with `std + 1e-5`. Saved physical observations can differ slightly from the
original NetCDF values because they passed through float32 batch tensors.
Model outputs are not automatically clipped to nonnegative runoff.

See [outputs](../reference/outputs.md#array-shapes),
[metrics](../reference/metrics.md), and the versioned
[benchmark records](../reference/benchmarks.md).
