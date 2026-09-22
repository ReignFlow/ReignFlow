# Choose a model

Start with **LSTM** to check a new dataset. Choose another model to investigate
a specific scientific question while keeping the data and evaluation protocol
consistent.

| Model | When to use it | Inputs and requirements |
|---|---|---|
| [LSTM](lstm.md) | First rainfall–runoff baseline | Daily meteorological forcing; optional basin attributes; CPU or CUDA |
| [Transformer](transformer.md) | Compare attention with recurrent modeling | Daily forcing and optional attributes; calendar features are supplied automatically; CPU or CUDA |
| [dHBV](dhbv.md) | Learn parameters of a differentiable HBV simulation | Precipitation, temperature, PET, and runoff in the required physical units; CPU or CUDA |

For a first neural run, keep `MaskedMSE`, the tutorial's small model, and a
constant learning rate. Basin-weighted loss is an optional comparison once
the baseline works. dHBV instead requires `CAMELS_dHBV`, `CompositeRMSE`, and
its documented warm-up and routing settings.

## Simulation or forecasting

| Goal | Task | Supported models |
|---|---|---|
| Estimate runoff on days covered by the input forcing | `regression` | LSTM, Transformer, dHBV |
| Estimate runoff after the input window | `forecast` | LSTM, Transformer |

For neural models, use `0 < pred_len <= seq_len`. Regression targets occupy
the last `pred_len` input days; forecast targets follow those inputs. See the
[window example](../guides/data-selection.md#periods-and-history).
Forecasting currently means a supervised experiment with observations for
evaluation; the [forecast guide](../guides/forecast.md) explains its outputs.

The supplied dHBV test path uses a long physical warm-up and supports
regression simulation. A parsed forecast flag does not provide a dHBV
forecasting workflow.

## Inputs and outputs

Neural models learn normalized runoff, which is converted back to physical
units for reporting. dHBV directly produces physical runoff. Model developers
can find tensor shapes and adapter requirements in the
[batch-field reference](../development/model-integration.md#batch-fields).

<details markdown="1">
<summary>Developer reference: model registry entries</summary>

The following table is generated from the source registry for the models
covered by this guide.

<!-- BEGIN GENERATED: models -->

| Model | Prediction space | Test path | Required loss |
|---|---|---|---|
| `LSTM` | `normalized` | `window` | No registry-enforced loss |
| `Transformer` | `normalized` | `window` | No registry-enforced loss |
| `dHBV` | `physical` | `dhbv_long_warmup` | `CompositeRMSE` |

<!-- END GENERATED: models -->

</details>
