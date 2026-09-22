# User guide

Use these pages to understand the data requirements and configure an
experiment. Start with the [quick start](../getting-started/quickstart.md)
for your first run, or choose a [complete example](../tutorials/index.md).

| Task | Read next |
|---|---|
| Prepare observations | [CAMELS](../data/camels.md) or [your own daily data](../data/custom-data.md) |
| Choose inputs and periods | [Variables and dates](data-selection.md), [station selection](station-selection.md) |
| Select a model | [LSTM, Transformer, and dHBV](../models/overview.md) |
| Train and evaluate | [Experiment workflow](training.md), [training controls](training-control.md), [validation](validation-and-model-selection.md) |
| Forecast beyond the input window | [Forecasting](forecast.md) |
| Continue or reuse a run | [Resume and warm start](resume-and-warm-start.md), [evaluate and share](evaluate-and-share.md) |
| Run on a cluster | [GPU jobs on Slurm](hpc.md) |
| Investigate a problem | [Troubleshooting](../reference/troubleshooting.md) |

For exact defaults, metric definitions, and file layouts, use the
[reference](../reference/index.md). Keep the same data and evaluation protocol
when comparing models; [benchmark context](../reference/benchmarks.md)
explains the historical records.
