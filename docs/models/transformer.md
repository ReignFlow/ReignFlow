# Transformer

Self-attention compares days within the input window so the model can combine
information from distant rainfall events directly.

`--model Transformer` uses a linear forcing/attribute embedding, a linear
embedding of daily calendar features, and stacked attention encoder layers.
The encoder uses ALiBi distance biases, pre-layer normalization, residual
connections, and feed-forward blocks. A final normalization, GELU, and linear
head produce normalized runoff at every input position.

## Research context

The implementation follows the base hydrologic Transformer studied by
[Liu et al. (2024)](https://doi.org/10.1016/j.jhydrol.2024.131389), with the
ALiBi and pre-layer normalization changes described above.
[Liu et al. (2025)](https://doi.org/10.5194/hess-29-6811-2025) provides broader
architecture comparisons. Use the [current model overview](overview.md) for
the architectures and tasks supported by this checkout.

## Configuration

| Option | Default | Meaning |
|---|---:|---|
| `--d_model` | 256 | Embedding width; must be divisible by the head count |
| `--transformer_n_layers` | 4 | Encoder layers |
| `--transformer_n_heads` | 4 | Attention heads |
| `--transformer_d_ff` | unset | Feed-forward width, resolved to `2 * d_model` |
| `--dropout` | 0.1 | Embedding and encoder dropout |
| `--transformer_weight_dropout` | 0.0 | DropConnect on Q/K/V projections |
| `--transformer_compile` | off | Wrap the encoder with `torch.compile` |

The historical benchmark uses width 128; that is a recipe override, not the
CLI default. Calendar features are supplied automatically as day-of-week,
day-of-month, and day-of-year in approximately `[-0.5, 0.5]`. The adapter
requires `batch_x_time_stamp`; no additional NetCDF calendar variables are needed.

## Run a CPU example

Follow the [Transformer example](../tutorials/transformer.md) for data creation,
the complete training command, and output checks.

## Attention and forecasting

Attention is bidirectional within the supplied input window; the distance
bias is not a causal mask. With regression targets spanning several input
days, an earlier target can therefore see later forcing within that window.
Use `pred_len=1` for the usual last-day simulation comparison. In the
[forecast task](../guides/forecast.md), all targets lie after the input window.

The current adapter uses the last `pred_len` encoder outputs as the forecast
vector. It has no autoregressive decoder, supplied future-weather input, or
explicit future-calendar decoder input.

## Checkpoint limitation

Transformer-specific settings are recorded in the effective configuration,
but the current model/data and exact-resume hash field lists omit the
`transformer_*` options. Tensor shape checks detect some changes, such as a
different feed-forward width, but can miss behavioral changes such as head
count or weight dropout. **Keep all Transformer settings identical when
replaying or resuming, and inspect `configs.sh`; a matching hash alone is
insufficient.** This documentation does not change checkpoint semantics.

Use matching compilation settings as well: `torch.compile` can change state
dictionary key structure. Compiled/eager checkpoint interchange is not a
promised compatibility path.
