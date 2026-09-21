# Quick start

Run these blocks in order in **one Bash session**, from the repository root
with your ReignFlow environment active. They create three synthetic basins and
exercise training, validation, checkpoint evaluation, and bundle loading on CPU.
No CAMELS download is required. The resulting scores describe artificial data.

## 1. Create the demonstration data

<!-- example: demo-data -->
```bash
python examples/quickstart/create_demo_data.py --output output/demo/CAMELS.nc
```

The file contains daily forcing and runoff from December 1999 through August
2000, two static attributes, and string station IDs. Names match the CAMELS
schema, but every value is generated for this example.

## 2. Train and select a checkpoint

The Bash array keeps the same data and model settings for later evaluations.
Training uses January–March, validation April, and testing May–June.

<!-- example: quickstart-train -->
```bash
demo_args=(
  --task_name regression --model LSTM --data CAMELS
  --input_nc_file output/demo/CAMELS.nc --all_stations
  --time_series_variables daymet_prcp,daymet_tmean,daymet_pet
  --static_variables area_gages2,elev_mean
  --train_date_list 2000-01-01,2000-03-31
  --val_date_list 2000-04-01,2000-04-30
  --test_date_list 2000-05-01,2000-06-30
  --seq_len 14 --pred_len 1 --d_model 16 --dropout 0
  --batch_size 32 --epochs 2 --learning_rate 0.001
  --do_eval --device cpu --seed 42
)
python -m reignflow "${demo_args[@]}" \
  --save_best --export_inference_bundle \
  --output_dir output/demo/lstm --des quickstart

run_dir="$(python -c 'from pathlib import Path; print(max(Path("output/demo/lstm").glob("regression_LSTM_*"), key=lambda p: p.name))')"
printf '%s\n' "$run_dir"
```

The run prints training loss, validation loss and median NSE, then test NSE,
KGE, correlation, and RMSE. `--save_best` chooses the highest finite median
validation NSE and restores those weights for the final test. Scores and
timings depend on the software and hardware; a particular score is not an
installation requirement.

The run directory contains `checkpoints/best.pt`, the latest numbered epoch,
`results/pred.npy`, `results/true.npy`, and `inference_bundle/`. The two physical
result arrays have shape **`[3, 61, 1]`**: three basins, 61 test days, one target.
See [run outputs](../reference/outputs.md) for the other files.

## 3. Evaluate the saved checkpoint

<!-- example: quickstart-replay -->
```bash
python -m reignflow "${demo_args[@]}" --do_test \
  --resume_from_checkpoint "$run_dir" --checkpoint_selector best \
  --output_dir output/demo/replay --des replay
```

This creates a separate evaluation run. Keep `--do_eval` in `demo_args`:
the loaded validation split contributes to the checkpoint's data fingerprint.
Use `.pt` files only from a trusted source.

## 4. Load the exported inference bundle

<!-- example: quickstart-bundle -->
```bash
python -m reignflow "${demo_args[@]}" --do_test \
  --inference_bundle "$run_dir/inference_bundle" \
  --output_dir output/demo/bundle-replay --des bundle-replay
```

The bundle uses safetensors and numeric/JSON sidecars. This command evaluates
the same selected weights and canonical data as the checkpoint command. The
documentation test compares their predictions, observations, and metrics.
Bundles currently require matching data, including target observations;
see [evaluation and sharing](../guides/evaluate-and-share.md).

## 5. Inspect the predictions

<!-- example: quickstart-metrics -->
```bash
python - "$run_dir" <<'PY'
from pathlib import Path
import sys
import numpy as np
from reignflow.utils.stats.metrics import cal_stations_metrics

run = Path(sys.argv[1])
pred = np.load(run / "results/pred.npy", allow_pickle=False)
true = np.load(run / "results/true.npy", allow_pickle=False)
print("Prediction shape:", pred.shape)
metrics = cal_stations_metrics(true[:, :, 0], pred[:, :, 0], ["NSE", "RMSE"])
print({name: float(np.nanmedian(values)) for name, values in metrics.items()})
PY
```

Continue with [real CAMELS training](../tutorials/first-lstm.md),
[Transformer](../models/transformer.md), or [dHBV](../tutorials/dhbv.md).
Keep synthetic outputs separate from scientific experiment results.
