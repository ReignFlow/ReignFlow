# LSTM models

## LSTM

`--model LSTM` concatenates normalized forcing and repeated basin attributes,
runs a one-layer `torch.nn.LSTM`, then applies output dropout and a linear
projection to the target channels. The model returns the full input-length
sequence; the trainer uses its final `pred_len` steps.

| Setting | Meaning |
|---|---|
| `--d_model` | Recurrent hidden size; default 256 |
| `--dropout` | Dropout after the recurrent sequence; default 0.1 |
| `--initial_forget_bias` | Value assigned to the recurrent forget-gate bias slice; default 3.0 |

The plain LSTM runs on CPU or CUDA. Begin with the [quick start](../getting-started/quickstart.md)
and use the [CAMELS tutorial](../tutorials/first-lstm.md) for observations.

## LSTM_mask

`--model LSTM_mask` wraps a cuDNN LSTM adapted from hydroDL-style code.
`--dropout` controls DropConnect on input and recurrent weight matrices during
training, rather than the plain LSTM's output dropout. The adapter calls
`torch._cudnn_rnn` and requires CUDA. Its CPU failure explicitly says
`requires a CUDA GPU`.

Start from an LSTM data configuration, choose `--model LSTM_mask`, and run
inside a GPU allocation with `--device cuda:0`. Treat the model change as a
new experiment; plain-LSTM weights are not interchangeable with these weights.

## Losses and units

Both neural models predict normalized targets. `MaskedMSE` averages squared
errors over non-NaN observations. `BasinNormalizedMSE` additionally weights each
error using the basin's standard deviation from the **raw loaded training
targets**:

```text
loss = mean((z_prediction - z_observation)^2 / (basin_std + 0.1)^2)
```

The mean is over the valid target elements. `basin_std` uses NumPy's population
standard deviation; it is distinct from the global target standard deviation
used by the z-score transform.

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
