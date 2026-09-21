# ReignFlow

!!! info "Documentation preview"

    This site is available for review before the public code release. Its
    repository contains documentation only. Model code, helper scripts, datasets,
    and trained weights are not included. The training and evaluation commands
    describe the full ReignFlow source checkout.

ReignFlow trains and evaluates daily rainfall–runoff models from NetCDF data.
Use LSTM as a starting point, compare attention with Transformer, or learn
parameters for a differentiable HBV simulation.

## Choose your starting point

| Your goal | Start here | What you will get |
|---|---|---|
| Run a model for the first time | [Installation](getting-started/installation.md) → [Quick start](getting-started/quickstart.md) | A small CPU run and an explained prediction plot |
| Use real observations or your own basins | [CAMELS tutorial](tutorials/first-lstm.md) or [custom data](data/custom-data.md) | A complete data-to-results workflow |
| Compare models or extend the framework | [Experiment workflow](guides/training.md) → [validation record](reference/validation.md) | Supported scenarios, comparison requirements, and the scope of checked results |

## Start with a complete run

The [quick start](getting-started/quickstart.md) has three steps: create
synthetic data, train an LSTM, and inspect predictions. You can read the
example plot now; running the commands requires the full source checkout.
After that, move to real data or [reload and share a model](guides/evaluate-and-share.md).

## Choose a model

| Model | Main use | Device |
|---|---|---|
| [LSTM](models/lstm.md) | First rainfall–runoff baseline | CPU or CUDA |
| [LSTM_mask](models/lstm.md#lstm_mask) | Recurrent model with weight DropConnect | CUDA |
| [Transformer](models/transformer.md) | Attention over forcing, attributes, and calendar features | CPU or CUDA |
| [dHBV](models/dhbv.md) | Neural parameter learning with a physical HBV simulation | CPU or CUDA |

[Model selection](models/overview.md) explains the input requirements and
supported tasks. [Experiment scenarios](guides/training.md#choose-your-workflow)
clarify forecasting, resuming, and the current limits of model reuse.

## Results and reproducibility

Use [run outputs](reference/outputs.md) to find saved arrays and plots, and
[troubleshooting](reference/troubleshooting.md#the-run-finishes-but-results-look-wrong)
when a completed run looks wrong. The [validation record](reference/validation.md)
identifies the checked source snapshot, environment, and executable examples.
[Historical benchmarks](reference/benchmarks.md) retain their original numerical
conventions and require separate reproduction before use as current scores.

## Acknowledgements

ReignFlow builds on [hydroDL](https://github.com/mhpi/hydroDL), its
[dPL-HBV implementation](https://github.com/mhpi/dPLHBVrelease), selected
[NeuralHydrology](https://github.com/neuralhydrology/neuralhydrology) metrics,
and other upstream work. See [acknowledgements and code provenance](acknowledgements.md)
for component sources, modifications, and scientific references.
