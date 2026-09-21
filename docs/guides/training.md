# Experiment workflow

A normal invocation loads training and test data, optionally loads validation,
trains, chooses weights, and tests. `--do_test` skips training but still loads
the data needed for compatibility checks. There is no separate train-only CLI
mode.

```text
CLI + dataset profile
        ↓
stations, variables, dates → training scaler → model windows
        ↓
training → optional validation → last / best / SWA weights
        ↓
held-out test → arrays, metrics, provenance, optional inference bundle
```

## Choose the operation

| Goal | Arguments added to the model/data configuration | Output location |
|---|---|---|
| Train a new model | No loading flag | New run |
| Report validation each epoch | `--do_eval` | Same training run |
| Select by validation NSE | `--save_best` | Same training run |
| Early stopping | `--patience N` | Same training run |
| Average final epochs | `--swa --swa_last_n N` | Same training run |
| Continue an interrupted run | `--resume_from_checkpoint RUN --checkpoint_selector latest` | Original run |
| Initialize a new experiment from weights | `--warm_start_from_checkpoint RUN --checkpoint_selector latest` | New run |
| Evaluate a saved checkpoint | `--do_test --resume_from_checkpoint RUN --checkpoint_selector best` | New run |
| Evaluate a bundle | `--do_test --inference_bundle BUNDLE` | New run |
| Export selected test weights | `--export_inference_bundle` | Current run's `inference_bundle/` |

The three weight-loading sources are mutually exclusive. Directory selectors
are `latest`, `best`, and `epoch:N`; give `swa.pt` as an explicit file.

## A useful order of work

1. Validate the environment with the [synthetic quick start](../getting-started/quickstart.md).
2. Inspect [data and units](../data/camels.md), then select [stations](station-selection.md)
   and [periods](data-selection.md).
3. Choose a [model](../models/overview.md), [loss and schedule](training-control.md).
4. Decide whether [validation selects the tested weights](validation-and-model-selection.md).
5. Run a small real-data trial before allocating a long [GPU job](hpc.md).
6. Inspect [physical arrays and metrics](../reference/outputs.md) and retain the
   command, data identity, and software version with the result.

An experiment identity includes more than its model name. Changing forcing
order, normalization, selection, window alignment, loss, or precision changes
the comparison. Historical scores and corrected-code results are distinguished
in [benchmarks](../reference/benchmarks.md).
