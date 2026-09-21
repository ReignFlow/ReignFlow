# Evaluate and share models

The [quick start](../getting-started/quickstart.md) provides complete executable
checkpoint and bundle replay commands. This page explains which settings and
files must accompany them.

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
