# Evaluate and share models

Use this page after [training your first model](../getting-started/quickstart.md).
A checkpoint stores saved model state. An inference bundle packages selected
weights and metadata for evaluation with matching data.

**Current scope:** checkpoint and bundle evaluation require matching periods,
stations, variables, and observations. General prediction on new, unobserved
data is not provided by this interface.

## Replay the quick-start run

These commands reuse the trained LSTM from the quick start. Run them in one
Bash session from the same source checkout. The array below is simply a way
to reuse the same settings for the two evaluation commands.

### Keep the original settings

<!-- example: quickstart-replay-setup -->
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
run_dir="$(python -c 'from pathlib import Path; print(max(Path("output/demo/lstm").glob("regression_LSTM_*"), key=lambda p: p.name))')"
```

Keep `--do_eval`: training used validation to select weights, so evaluation
must load the same validation data for its compatibility check.

### Evaluate the saved checkpoint

<!-- example: quickstart-replay -->
```bash
python -m reignflow "${demo_args[@]}" --do_test \
  --resume_from_checkpoint "$run_dir" --checkpoint_selector best \
  --export_inference_bundle --output_dir output/demo/replay --des replay

bundle_run="$(python -c 'from pathlib import Path; print(max(Path("output/demo/replay").glob("regression_LSTM_*"), key=lambda p: p.name))')"
```

This evaluates `best.pt` and exports the evaluated weights into the new
evaluation run's `inference_bundle/` directory. It does not retrain the model.
Load `.pt` files only from a trusted source.

### Evaluate the exported bundle

<!-- example: quickstart-bundle -->
```bash
python -m reignflow "${demo_args[@]}" --do_test \
  --inference_bundle "$bundle_run/inference_bundle" \
  --output_dir output/demo/bundle-replay --des bundle-replay
```

Predictions, observations, and metrics should match the original selected
model under the same environment. The documented workflow test checks their
array and metric equality. Bundles use safetensors and numeric/JSON sidecars.

## Evaluate a checkpoint

Repeat the original model/data configuration and add:

```text
--do_test --resume_from_checkpoint path/to/run --checkpoint_selector best
```

Use `latest` for the newest numbered checkpoint, `epoch:N` for a retained
numbered file, or an explicit `.pt` path without a selector. A test invocation
writes a new run directory; it does not append results to the source run.

`--do_test` requires an explicit checkpoint or bundle. It still constructs the
training and test datasets, and validation when enabled. It cannot evaluate
without matching observations or infer arbitrary new data from weights alone.

### Repeat the run configuration

The loader checks the current format, fitted scaler, model/data contract,
canonical decoded-data fingerprint, and model tensor compatibility. Keep the
same stations and order, variable order, periods, model dimensions, window
alignment, and normalization/missing-value rules.

Also retain effective validation: if training used `--save_best`, evaluation
can use `--save_best` or `--do_eval` to load the same validation split. Omitting
it changes the input fingerprint. Inspect the source `configs.sh` instead of
reconstructing settings from memory.

Transformer-specific settings are a current exception to hash coverage; keep
them identical and compare the saved configuration explicitly. See
[the limitation](../models/transformer.md#checkpoint-limitation).

## Export the weights used for testing

Add `--export_inference_bundle` to a new training or evaluation command. The
export happens after successful testing and contains the selected weights:
last epoch, best validation, SWA, a loaded checkpoint, or a loaded bundle.

| File in `inference_bundle/` | Contents |
|---|---|
| `model.safetensors` | Model tensors |
| `manifest.json` | Schema, model identifier, selected-weight source |
| `scaler.npz` | Numeric fitted statistics, loaded without pickle |
| `provenance_portable.json` | Portable configuration and compatibility records |
| `stations.txt` | Ordered string station IDs |

The bundle contains no optimizer or scheduler state. DataParallel's `module.`
prefix is stripped. This does not guarantee interchange of compiled and eager
Transformer state dictionaries.

## Evaluate a bundle

Use the matching model/data configuration with:

```text
--do_test --inference_bundle path/to/inference_bundle
```

Do not also pass a checkpoint source or selector. The loader reads safetensors
and safe numeric/JSON sidecars, then validates station identity, scaler shapes,
data/contract hashes, and model state. It never falls back to a pickle file.

Bundles currently support **replay with matching canonical data**. A changed
period, station order, forcing set, or decoded value is rejected. They are not
a general new-basin or observation-free prediction API, and warm-start training
from a bundle is not implemented.

## What to share

Prefer the complete inference bundle for public weights. `.pt` loading uses
pickle support, so recipients must trust the source before loading a training
checkpoint. Keep full local `provenance.json` with the experiment; it can include
machine paths. Inspect the portable record before publishing because scientific
settings and station information are intentionally retained.

Include the source version, data preparation/selection, environment, and exact
evaluation command. An old-normalization checkpoint must not be relabeled as
a corrected result; see [compatibility](checkpoints.md#normalization-compatibility).
