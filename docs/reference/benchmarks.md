# Benchmarks and reproduction

Benchmark results belong to a particular source version, data selection,
training recipe, and numerical environment. A passing tutorial establishes
that a workflow runs; it does not reproduce a full benchmark.

The [validation record](validation.md) describes the current documentation
checks separately from these historical scientific runs.

## Historical records

The following rounded scores are retained from the repository's existing
benchmark registry. **They predate the inverse-normalization correction and
are not newly verified scores for the current code.** Some records additionally
predate the dPL-to-dHBV identifier change.

**Cross-row comparability has not been established.** The rows use different
forcing inputs and basin sets, including 531 versus 671 basins. Before ranking
models, match stations, dates, forcings, loss, weight selection, normalization,
and seed protocol. The table is an inventory of recorded experiments.

| Model/recipe | Basins | Median NSE | Record |
|---|---:|---:|---|
| LSTM, 18 forcings, milestone schedule | 531 | 0.8014 | Independently verified on its archived stack |
| LSTM, Daymet | 531 | 0.7359 | Recorded |
| LSTM_mask, Daymet | 531 | 0.7485 | Recorded |
| LSTM, 18 forcings, cosine schedule | 531 | 0.7966 | Recorded |
| dHBV, Daymet P/T/PET | 671 | 0.7162 | Historical physical-output baseline |
| Transformer, 18 forcings | 531 | 0.7659 | Recorded single run; separate three-seed mean 0.7712 |

Full-precision metrics, configurations, and the original verification history
remain in `examples/benchmark/BENCHMARKS.md` in the full development checkout;
the registry and experiment artifacts are not included in this documentation
preview. Those records should be read with the version qualification above.
The preview does not change or replace the original experiment artifacts.

## Run an existing recipe

Prepare the full CAMELS file at the profile default path, activate the project
environment, and obtain a suitable GPU allocation. From the repository root,
choose **one** recipe:

```bash
bash examples/benchmark/lstm_benchmark.sh
```

```bash
bash examples/benchmark/dhbv_benchmark.sh
```

```bash
bash examples/benchmark/transformer_benchmark.sh
```

The LSTM script's first recipe is active; other variants are commented out.
The dHBV script inherits automatic device selection, so verify that CUDA is
visible before launching it. These are long scientific runs, not the CPU
documentation tests.

The Transformer recipe uses width 128 and CUDA AMP/TF32/compilation overrides;
these are not its parser defaults. Keep its full settings for replay because
of the [current hash-coverage limitation](../models/transformer.md#checkpoint-limitation).

## Interpret normalization changes

Neural physical predictions and observations now use
`z * (std + 1e-5) + mean`. Old outputs used `z * std + mean`. This can change
physical metrics and, near zero, which observations survive the reporting
mask. dHBV's physical-output branch bypasses the inverse, but its compatibility
metadata can still differ between versions.

Historical full-reproduction jobs, including those performed during server
migration, do not automatically verify later numerical fixes. Re-evaluate and
record corrected outputs before quoting them as current benchmark results.
Do not relabel old artifacts or weaken checkpoint checks to obtain a match.

## Reproduction checklist

1. Record the commit and any local changes; retain the exact source used.
2. Record the dataset preparation, ordered stations and variables, dates,
   missing-value rules, scaler contract, and decoded-data fingerprint.
3. Retain the complete command, including loss, schedule, sampler, seed,
   worker count, precision, compilation, and selected test-weight source.
4. Record Python/PyTorch/CUDA/cuDNN and hardware.
5. Retain predictions, observations, metrics, checkpoints, and provenance.
6. Compare a second run with declared tolerances and explain differences;
   distinguish a recorded run from an independently reproduced result.

Validation-selected and last-epoch results answer different questions. Do not
enable `--save_best` on a recipe whose validation interval overlaps testing.
For quick implementation checks, use the [real-data gate](../development/testing.md#real-data-gate)
before committing resources to a full reproduction.
