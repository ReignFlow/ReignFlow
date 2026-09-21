# Transformer

`--model Transformer` uses a linear forcing/attribute embedding, a linear
embedding of daily calendar features, and stacked attention encoder layers.
The encoder uses ALiBi distance biases, pre-layer normalization, residual
connections, and feed-forward blocks. A final normalization, GELU, and linear
head produce normalized runoff at every input position.

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

First create the [quick-start data](../getting-started/quickstart.md#1-create-the-demonstration-data).
Then run this complete command:

<!-- example: transformer-train -->
```bash
python -m reignflow --task_name regression --model Transformer --data CAMELS \
  --input_nc_file output/demo/CAMELS.nc --all_stations \
  --time_series_variables daymet_prcp,daymet_tmean,daymet_pet \
  --static_variables area_gages2,elev_mean \
  --train_date_list 2000-01-01,2000-03-31 \
  --val_date_list 2000-04-01,2000-04-30 \
  --test_date_list 2000-05-01,2000-06-30 \
  --seq_len 14 --pred_len 1 --d_model 16 \
  --transformer_n_layers 1 --transformer_n_heads 4 --transformer_d_ff 32 \
  --dropout 0 --batch_size 32 --epochs 1 --learning_rate 0.001 \
  --do_eval --device cpu --seed 42 --output_dir output/demo/transformer
```

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
