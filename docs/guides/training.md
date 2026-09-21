# Experiment workflow

Choose the workflow that matches your question before selecting extra training
options. The [quick start](../getting-started/quickstart.md) is the baseline
for a first successful run.

## Choose your workflow

| What you want to do | Current support | Start here |
|---|---|---|
| Train on your own daily observations | Named NetCDF variables, observed targets, and explicit station/date selections | [Custom-data example](../data/custom-data.md) |
| Compare LSTM, Transformer, or dHBV | Match the data and evaluation protocol; model-specific physical inputs and warm-up can differ | [Model selection](../models/overview.md), [benchmark context](../reference/benchmarks.md) |
| Forecast after the input window | Supervised neural forecast experiment; reported daily metrics mix lead times | [Forecasting](forecast.md) |
| Continue interrupted training | Exact resume keeps the original training plan; warm start begins a changed plan from saved weights | [Resume or warm start](resume-and-warm-start.md) |
| Reload and share a trained model | Checkpoint/bundle replay with matching data, including observations | [Evaluate and share](evaluate-and-share.md) |
| Apply a bundle to unseen basins or an observation-free future period | No general prediction API for this workflow in the current release preview | [Reuse limitations](evaluate-and-share.md#evaluate-a-bundle) |
| Add a model | Explicit adapter and registry entry | [Model integration](../development/model-integration.md) |

## First baseline, then comparisons

1. Check the [synthetic example](../getting-started/quickstart.md), then inspect
   your real data's dates, units, variables, and missing observations.
2. Select disjoint training, validation, and test periods. Earlier history may
   be needed to supply complete [input windows](data-selection.md#periods-and-history).
3. Start with the LSTM tutorial's settings and `--save_best` for validation
   selection. Inspect one basin's prediction plot and the basin-wise metrics.
4. Change one scientific choice at a time. Keep a record of the command, data
   selection, environment, and selected weights for each comparison.

For an existing benchmark, use its full recipe. Enabling validation selection
or changing the data means a different experiment even when the model name
is unchanged.

## Choose the operation

A normal invocation trains, selects weights, and tests. `--do_test` skips
training but still loads the data required for evaluation and compatibility
checks. There is no separate train-only CLI mode.

| Operation | Arguments added to the model/data configuration | Output |
|---|---|---|
| New training | No loading flag | New run |
| Validation-selected training | `--save_best` | Best validation weights used for the final test |
| Resume | `--resume_from_checkpoint RUN --checkpoint_selector latest` | Continue in the original run |
| Warm start | `--warm_start_from_checkpoint RUN --checkpoint_selector latest` | New training run |
| Evaluate checkpoint | `--do_test --resume_from_checkpoint RUN --checkpoint_selector best` | New evaluation run |
| Evaluate bundle | `--do_test --inference_bundle BUNDLE` | New evaluation run |
| Export selected weights | `--export_inference_bundle` | Current run's `inference_bundle/` after testing |

The three weight-loading sources are mutually exclusive. Directory selectors
are `latest`, `best`, and `epoch:N`; give `swa.pt` as an explicit file.
See [training controls](training-control.md) for optimization, sampling, and
devices, and [model selection](validation-and-model-selection.md) for early
stopping and weight averaging. Long experiments can use [GPU jobs](hpc.md).
