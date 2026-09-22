# Train a Transformer

Train a small attention model on three synthetic basins and inspect its saved
runoff predictions. Read the [model guide](../models/transformer.md) for the
architecture and [configuration reference](../reference/cli.md#model-and-window-settings)
for available options.

## Before you start

Complete [installation](../getting-started/installation.md) and run from the full
source checkout with the environment active. This example uses CPU execution.

## 1. Prepare the data

```bash
python examples/quickstart/create_demo_data.py --output output/demo/CAMELS.nc
```

The [demonstration file](../getting-started/quickstart.md#1-create-the-demonstration-data)
contains the forcing, attributes, and artificial runoff used below.

## 2. Train and evaluate

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

The model uses 14 input days to estimate runoff on the final day. The command
trains on January–March, reports validation on April, and tests May–June.
It evaluates the final weights; it does not request validation-based weight
selection. One epoch is a workflow check, not a benchmark recipe.

## 3. Inspect the result

Follow the printed directory under `output/demo/transformer/`. The
`results/pred.npy` and `results/true.npy` arrays have shape `[3, 61, 1]`,
covering 1 May–30 June 2000 in mm/day. The program reports basin-median metrics
and saves `results/feature_Runoff.png` for the first basin.

Use the [quick-start interpretation](../getting-started/quickstart.md#3-inspect-the-predictions)
to read the metrics, and [run outputs](../reference/outputs.md) to locate the
configuration and saved arrays.

## Next steps

Use [your own observations](../data/custom-data.md) or compare with the
[LSTM example](first-lstm.md) under a matched data and training protocol.
Keep every Transformer setting unchanged for
[checkpoint replay](../models/transformer.md#checkpoint-limitation).
