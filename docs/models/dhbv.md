# Differentiable HBV

`--model dHBV` generates basin-specific parameters with an LSTM, then runs
multi-component HBV and gamma routing. It implements the dPL-HBV approach
through a concrete ReignFlow adapter and physical-variable contract.

## Data flow

```text
normalized forcing + normalized attributes → parameter LSTM
                                                ↓
raw precipitation / temperature / PET → HBV components → routing → runoff
```

The LSTM's last output generates 12 HBV parameters per component and two
routing parameters. Sigmoid outputs are mapped to physical parameter ranges.
The default CAMELS_dHBV profile uses 24 components and uniform weights;
the model averages component discharges, not component parameter values.

The model returns runoff in physical units. Training uses `raw_batch_y`,
and testing bypasses inverse normalization. Normalized forcing and attributes
are used only to generate parameters.

## Window and routing requirements

Use `--task_name regression --data CAMELS_dHBV --criterion CompositeRMSE`.
The model requires:

```text
seq_len - warmup_days == pred_len
```

Warm-up is inside the requested window. Each constructed train, test, or
enabled validation split must fit `seq_len` days. The standard routed profile
also requires **at least 15 output steps** because the routing kernel has
15 lags. A short example must satisfy both constraints, such as
`seq_len=30`, `warmup_days=15`, `pred_len=15`.

The production window is 730/365/365. `--nmul` changes the component count;
`--d_model`, `--dropout`, and `--initial_forget_bias` configure the parameter LSTM.
The [tutorial](../tutorials/dhbv.md) supplies a runnable CPU example.

## Physical forcing

The profile maps precipitation, temperature, PET, and runoff to
`daymet_prcp`, `daymet_tmean`, `daymet_pet`, and `Runoff`. The reader creates
`physics_batch_x` in fixed P/T/PET order even when the general normalized
forcing list is reordered. All required physical variables must remain in
the selected forcing list.

P, PET, and runoff use `mm day-1`; temperature uses `degC`. Declared
incompatible units fail rather than being converted. Missing metadata can use
the explicit profile declaration with a warning.

Maintain finite physical forcing for training. The normalized branch's NaN-to-zero
handling does not impute the raw physical training inputs. Long-warm-up testing
has its own explicit warning and zero-fill for raw forcing NaNs, while infinite
forcing fails. A test-time fill is not a scientifically neutral correction to
missing precipitation or temperature.

## Loss

The required CompositeRMSE loss sums across target channels:

```text
(1 - alpha) * RMSE(q_prediction, q_observation)
  + alpha * RMSE(log10(sqrt(q_prediction + 1e-6) + 0.1),
                 log10(sqrt(q_observation + 1e-6) + 0.1))
```

`alpha` is `--loss_alpha`, default 0.25. NaN observations are masked; the
transformed branch also excludes values outside its square-root domain.
Exact zero error has a finite zero gradient. The standard adapter outputs
one discharge channel.

## Test-time long warm-up

The parameter LSTM reads the complete normalized training period once per
basin. HBV then sees the concatenated physical training and test forcing;
training is the warm-up and only the test interval is reported. Therefore,
test must begin exactly one day after training ends. Validation instead uses
training-style internal-warm-up windows.

Testing processes basins in fixed batches of 50 in this implementation, not
the CLI training batch size. The long-warm-up path unwraps DataParallel and
runs the core on its primary device; it is not distributed over multiple GPUs.

CPU supports small checks; CUDA is useful for long runs. `--use_amp` enables
autocast around the parameterization path during training and long-warm-up
test, while HBV casts its physical computation to fp32. Ordinary validation
remains fp32. Compilation and hardware still require numerical verification.

## Parameter export

After successful testing, `results/hbv_parameters.nc` and its JSON sidecar
record the generated component ensemble and routing kernel from the selected
test weights. Use `--export_hbv_parameters False` to disable them.
[HBV parameter files](../guides/hbv-parameters.md) documents the dimensions,
units, and reuse constraints.
