# Choose a model

The following table is generated from `MODEL_REGISTRY` in the current source.

<!-- BEGIN GENERATED: models -->

| Model | Prediction space | Test path | Required loss |
|---|---|---|---|
| `LSTM` | `normalized` | `window` | No registry-enforced loss |
| `LSTM_mask` | `normalized` | `window` | No registry-enforced loss |
| `Transformer` | `normalized` | `window` | No registry-enforced loss |
| `dHBV` | `physical` | `dhbv_long_warmup` | `CompositeRMSE` |

<!-- END GENERATED: models -->

## Practical choices

- **LSTM** is the simplest CPU or CUDA baseline. Start here to verify your data.
- **LSTM_mask** uses a CUDA-only cuDNN cell with weight DropConnect.
- **Transformer** uses attention and daily calendar features. It supports CPU
  smoke runs and CUDA training.
- **dHBV** generates parameters for an HBV simulation and returns physical runoff.
  Use `CAMELS_dHBV`, `CompositeRMSE`, and the documented warm-up settings.

Use `MaskedMSE` or `BasinNormalizedMSE` for the neural models. The absence of a
registry-enforced loss is not evidence that every CLI loss is a useful pairing:
the standard neural path passes basin statistics to its loss, while dHBV uses
raw physical targets.

## Inputs and outputs

All adapters receive a batch dictionary. `B`, `T`, `P`, `F`, `C`, and `Y` denote
batch size, input steps, target steps, forcing channels, static channels, and
target channels.

| Field | Shape | Available in the standard Dataset |
|---|---|---|
| `batch_x` | `[B, T, F]` | Normalized forcing |
| `batch_c` | `[B, C]` | Normalized attributes; zero-width if disabled |
| `batch_y` | `[B, P, Y]` | Normalized observations |
| `raw_batch_y` | `[B, P, Y]` | Physical observations |
| `batch_x_time_stamp` | `[B, T, 3]` | Daily calendar features |
| `batch_y_time_stamp` | `[B, P, 3]` | Target calendar features |
| `batch_target_std` | `[B, Y]` or empty | Raw training-period basin standard deviations for `BasinNormalizedMSE` |
| `physics_batch_x` | `[B, T, 3]` | Raw P/T/PET when the profile defines physical roles |

The Dataset does **not** supply a general `raw_batch_x` field. Full raw arrays
remain in `Dataset.data_dict_all`; adapters should consume the documented batch
fields rather than assume raw forcing is always present.

Every model returns `{"outputs_time_series": prediction}`. The trainer scores
the final `pred_len` output steps. Neural models concatenate repeated static
attributes with forcing; Transformer also consumes the input calendar features.
dHBV keeps its normalized parameterization inputs separate from physical forcing.

## Tasks

For `regression`, targets occupy the final part of the input window. For
`forecast`, targets follow the input window. Use `0 < pred_len <= seq_len` for
the current neural adapters: they return one output per input step and the
trainer takes the final `pred_len` outputs.

The dHBV long-warm-up test is a regression simulation. The general CLI may parse
`forecast` with dHBV, but it does not provide a supported physical forecast
workflow. Use [neural forecasting](../guides/forecast.md) for that task.

Read the model-specific pages for [LSTM](lstm.md),
[Transformer](transformer.md), and [dHBV](dhbv.md).
