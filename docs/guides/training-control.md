# Configure training

Begin with the defaults used by a complete tutorial, then change one setting
at a time. The [CLI reference](../reference/cli.md) is generated from the parser;
this page explains effects that a default-value table cannot capture.

## What to change first

| Stage | Settings to consider |
|---|---|
| Before using your own data | File path, variables, stations, units, and split dates |
| First LSTM baseline | Keep the tutorial's small model, `MaskedMSE`, AdamW, constant learning rate, and fp32; use `--save_best` with disjoint validation |
| Controlled experiments | Change window length, loss, model size, scheduler, or sampler one at a time; evaluate AMP, compilation, and SWA after the baseline works |

These are starting choices, not a claim of optimal accuracy. Published
benchmark comparisons require their complete recorded recipe.

## Optimizer and loss

The optimizer registry contains `AdamW`, `SGD`, and `Adadelta`. All receive
`--learning_rate` and `--weight_decay`. `--clip_grad` optionally clips the
global gradient norm; its default is unset.

| Criterion | Intended use | Calculation |
|---|---|---|
| `MaskedMSE` | Neural models; default | Mean squared normalized error over non-NaN targets |
| `BasinNormalizedMSE` | Neural basin-weighted training | Normalized squared error divided by `(raw_basin_std + 0.1)^2` |
| `CompositeRMSE` | Required for dHBV | Physical RMSE mixed with log/square-root-space RMSE |

`BasinNormalizedMSE` uses raw per-basin training target standard deviations.
There is no implemented `target_unit`/`freq` rescaling; see [the exact loss](../models/lstm.md#losses-and-units).
For `CompositeRMSE`, `--loss_alpha` is the transformed-space weight (default 0.25).

An all-NaN target batch skips the optimizer update. Nonfinite predictions,
losses, or gradients are handled explicitly; a run with no useful training or
validation targets fails rather than reporting a successful empty epoch.

## Learning-rate schedules

No scheduler means a constant learning rate.

| Scheduler | Advances | Behavior |
|---|---|---|
| `LSTMMilestoneLR` | Each epoch | Fixed rates: 0.001 in epochs 1–9, 0.0005 in 10–24, 0.0001 from 25 |
| `CosineAnnealingLR` | Each epoch | Decays over requested epochs to `lr_min`, or `learning_rate / 100` |
| `OneCycleLR` | Each successful optimizer update | Uses `max_lr`, `pct_start`, `div_factor`, `final_div_factor`, `anneal_strategy` |
| `ReduceLROnPlateau` | Each epoch with validation loss | Factor 0.5 and scheduler patience 5; requires validation |

The milestone schedule sets absolute rates; it does not scale an arbitrary
starting rate. OneCycle plans `epochs * batches_per_epoch` updates, so changing
sampling or batch size also changes its schedule. NaN/AMP-overflow skipped
updates do not advance OneCycle. Early stopping can finish a schedule early.

Plateau monitors validation **loss**, while `--save_best` and run-level
`--patience` monitor median validation **NSE**. These are separate decisions.
Scheduler state is saved in numbered checkpoints for exact resume.

## Window sampling

`all_windows` shuffles and visits each candidate once per epoch, including a
possibly partial final batch. `random_windows` draws with replacement and can
repeat or omit individual windows. Its derived budget is:

```text
coverage = batch_size * pred_len / (n_basins * (loaded_train_days - warmup_days))
n_iters = ceil(log(0.01) / log(1 - coverage))
```

Coverage must be strictly below 1. Each random epoch has `n_iters` full batches.
This is an approximate basin-time coverage budget, not a guarantee that 99%
of unique windows appear. For small datasets, start with `all_windows`; a
production dHBV batch size often violates the random-window budget on a few basins.

```text
sample_len = seq_len                     regression
sample_len = seq_len + pred_len          forecast
windows_per_basin = loaded_days - sample_len + 1
```

The startup `[sampling]` line records candidate windows, replacement, sampled
windows, batches, and planned update attempts. Actual successful updates can
be fewer. See [periods and history](data-selection.md#periods-and-history).

## Device and workers

| Value | Effect |
|---|---|
| `--device auto` | First visible CUDA device, or CPU |
| `--device cpu` | CPU |
| `--device cuda:0` | First GPU visible to this process |
| `--device cuda:0,1` | Single-process DataParallel on two visible GPUs |

The program does not rewrite `CUDA_VISIBLE_DEVICES`. Set GPU visibility before
Python starts, normally through the scheduler. DataParallel is a one-node
mode; the main CLI does not launch multi-node DDP. Independent experiments
can run as separate [Slurm array tasks](hpc.md).

`--num_workers 0` loads batches in the main process and is the documented
baseline. Increase workers only after measuring a benefit. Data arrays are
already loaded into memory; worker count does not make NetCDF streaming.

## Precision and compilation

- `--use_amp` enables CUDA autocast for training and dHBV long-warm-up testing.
  On CPU it records the request and uses fp32.
- Validation and ordinary neural testing use fp32 even after AMP training.
- `--allow_tf32` enables supported CUDA TF32 matrix operations.
- `--compile_hbv` compiles the dHBV training loop body; long-warm-up testing
  switches to its shape-independent TorchScript loop.
- `--transformer_compile` compiles the Transformer encoder.

These choices can change numerical results and startup cost. Record the flags,
GPU, PyTorch/CUDA versions, and source state. Do not promise bitwise equivalence
across devices, compilation modes, or precision settings. AMP checkpoints also
record the gradient scaler and successful/overflow-skipped update counts.

`--seed` sets Python, NumPy, and Torch random generators. Seeds support
controlled comparisons, but do not guarantee identical floating-point results
on different machines.
