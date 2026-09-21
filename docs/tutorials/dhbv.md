# Train differentiable HBV

dHBV uses a neural network to generate basin-specific HBV parameters and trains
through a physical simulation. Its targets and loss are in physical runoff
units. Start with the following small CPU run to check the whole path,
including the parameter export.

## Required alignment

```text
seq_len - warmup_days == pred_len
```

Warm-up is inside each input window. With 30 input days and 15 warm-up days,
the final 15 days contribute to the loss. Each constructed split must contain
at least 30 days. The routed profile also needs at least 15 output steps for
its fixed 15-day routing kernel: use `pred_len >= 15`. Satisfying the window
equation alone is insufficient for a shorter routed example. Production
settings such as 730/365/365 need correspondingly longer splits.

Final testing uses the complete training period as HBV warm-up. The test must
start exactly one day after the training end. This condition is checked at
test time, so check your dates before starting a long training run.

## Run a small example

Create the [quick-start data](../getting-started/quickstart.md#1-create-the-demonstration-data)
first. This example trains on January–March, tests April–June, and validates
on July–August; all periods are disjoint and the test follows training.

<!-- example: dhbv-train -->
```bash
python -m reignflow --task_name regression --model dHBV --data CAMELS_dHBV \
  --input_nc_file output/demo/CAMELS.nc --all_stations \
  --static_variables area_gages2,elev_mean \
  --train_date_list 2000-01-01,2000-03-31 \
  --val_date_list 2000-07-01,2000-08-31 \
  --test_date_list 2000-04-01,2000-06-30 \
  --criterion CompositeRMSE --seq_len 30 --warmup_days 15 --pred_len 15 \
  --nmul 2 --d_model 8 --dropout 0 --batch_size 32 \
  --epochs 1 --learning_rate 0.001 --sampling_strategy all_windows \
  --do_eval --device cpu --seed 42 --output_dir output/demo/dhbv
```

`CAMELS_dHBV` supplies P/T/PET roles and units. The example overrides static
attributes and station selection because the synthetic file is smaller than
the full profile. `all_windows` avoids the random-sampling budget constraints
of production batch sizes.

## Inspect the result

The physical test arrays have shape `[3, 91, 1]`. No inverse z-score transform
is applied to dHBV outputs. The run also writes:

```text
results/hbv_parameters.nc
results/hbv_parameters.metadata.json
```

The example's NetCDF has three stations, two components, 12 HBV parameters,
one routing component, two routing parameters, and 15 routing lags.
[HBV exports](../guides/hbv-parameters.md) explains each variable and how the
ensemble combines. Use `--export_hbv_parameters False` to disable these files.

## Move to CAMELS

Use a real CAMELS file with all profile variables and appropriate basin
selection. The production recipe uses 24 components and
`seq_len=730`, `warmup_days=365`, `pred_len=365`. Its batch size, loss,
random-window sampler, dates, and schedule are recorded in
[benchmarks](../reference/benchmarks.md).

CUDA is appropriate for long recursive simulations. A CPU synthetic pass
does not validate GPU/AMP behavior or reproduce a multi-decade experiment.
The [model reference](../models/dhbv.md) describes physical forcing and
long-warm-up inference; the [cluster guide](../guides/hpc.md) covers allocations.
