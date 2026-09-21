# ReignFlow

!!! info "Documentation preview"

    This site is available for review before the public code release. Its
    repository contains documentation only. Model code, helper scripts, datasets,
    and trained weights are not included. The training and evaluation commands
    describe the full ReignFlow source checkout.

ReignFlow trains and evaluates rainfall–runoff models from daily NetCDF data.
LSTM, LSTM_mask, Transformer, and differentiable HBV share a command-line
interface, data preparation, checkpoint handling, and basin-wise metrics.

ReignFlow builds on [hydroDL](https://github.com/mhpi/hydroDL), its
[dPL-HBV implementation](https://github.com/mhpi/dPLHBVrelease), and selected
[NeuralHydrology](https://github.com/neuralhydrology/neuralhydrology) metrics,
along with other upstream contributions. The
[acknowledgements](acknowledgements.md) identify each component's source and
ReignFlow's modifications.

## Start with a complete run

1. [Install ReignFlow](getting-started/installation.md).
2. [Run the quick start](getting-started/quickstart.md): create a small synthetic
   dataset, train on CPU, evaluate a checkpoint, and reload an inference bundle.
3. [Prepare CAMELS](data/camels.md) or [supply your own daily data](data/custom-data.md).
4. Follow the [LSTM tutorial](tutorials/first-lstm.md) or [dHBV tutorial](tutorials/dhbv.md).

The synthetic example checks the workflow. Scientific comparisons use real
observations, fixed selections, and the versioned [benchmark records](reference/benchmarks.md).

## Choose a model

| Model | What it learns | Device |
|---|---|---|
| [LSTM](models/lstm.md) | Runoff from a recurrent forcing and attribute encoder | CPU or CUDA |
| [LSTM_mask](models/lstm.md#lstm_mask) | Runoff with weight DropConnect in a cuDNN LSTM | CUDA |
| [Transformer](models/transformer.md) | Runoff from attention over forcing, attributes, and calendar features | CPU or CUDA |
| [dHBV](models/dhbv.md) | Basin parameters for a differentiable HBV simulation | CPU or CUDA |

[Model selection](models/overview.md) explains the output spaces, loss choices,
and supported workflows. Use daily data throughout; the current trainer has
daily history and calendar assumptions.

## Run an experiment

- [Select stations](guides/station-selection.md) and [configure variables and periods](guides/data-selection.md).
- [Configure training](guides/training-control.md), then [select weights with validation](guides/validation-and-model-selection.md).
- [Run GPU jobs on a cluster](guides/hpc.md).
- [Resume an interrupted run or transfer weights](guides/resume-and-warm-start.md).
- [Evaluate and export a model](guides/evaluate-and-share.md).
- [Run a forecast task](guides/forecast.md) or [inspect HBV parameters](guides/hbv-parameters.md).

The [workflow overview](guides/training.md) connects these steps. The
[CLI reference](reference/cli.md) lists the executable parser's defaults;
[outputs](reference/outputs.md), [metrics](reference/metrics.md), and
[troubleshooting](reference/troubleshooting.md) explain what a run produces.

## Documentation and reproducibility

These pages describe the development version reviewed before this preview was
exported. CLI, model, and profile tables were generated from code, and tutorial
commands were checked in the full source checkout. The preview build checks the
documentation site; it does not run the model tests.
[Validation coverage](development/documentation.md#validation-coverage)
distinguishes these checks from GPU testing and full benchmark reproduction.

The inverse normalization now uses `z * (std + 1e-5) + mean`. Historical
benchmark scores predate this correction unless their record states otherwise.
Checkpoints and inference bundles with the previous normalization contract
require an explicit migration decision; see [checkpoints](guides/checkpoints.md#normalization-compatibility).
