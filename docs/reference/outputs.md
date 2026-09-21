# Run outputs

New training, warm start, and standalone evaluation each create a new run.
Exact resume continues in the source run directory.

```text
output/<task>_<model>_<profile>_<seq_len>_<pred_len>_<description>__<UTC>_<hash>/
```

Use the path reported by the invocation. A model label alone is not a unique
experiment identifier.

## Files

| File | When written | Meaning |
|---|---|---|
| `checkpoints/checkpoint_epoch_N.pt` | Successful training epoch | Latest full training state; previous epoch normally removed |
| `checkpoints/best.pt` | Finite validation improvement with `--save_best` | Selected validation weights |
| `checkpoints/swa.pt` | SWA snapshots available | Averaged weights and selection metadata |
| `configs.sh`, `configs_input.sh` | New run setup | Effective and original commands |
| `provenance.json`, `provenance_portable.json` | New run setup | Full and portable experiment records |
| `results.txt` | During execution | Training/selection messages and aggregate metrics; not a copy of every console message |
| `results/loss_data.csv`, `loss_curve.png` | Training | Loss history and curve |
| `results/pred.npy`, `results/true.npy` | Successful test | Physical prediction and observation series |
| `results/pred_raw.npy`, `results/true_raw.npy` | Ordinary neural test | Normalized per-window arrays |
| `results/feature_<target>.png` | Test reporting | First basin's first up-to-1000 test steps for each target |
| `results/hbv_parameters.nc`, `results/hbv_parameters.metadata.json` | dHBV test with export enabled | HBV ensemble parameters and provenance |
| `inference_bundle/` | Test with `--export_inference_bundle` | Selected weights and safe sidecars |

An evaluation-only run does not train or create a new numbered training
checkpoint/loss history. Some setup files can exist even if a later stage fails;
file presence alone is not proof of successful testing.

## Array shapes

| Arrays | Shape | Space |
|---|---|---|
| `pred.npy`, `true.npy` | `[basin, test_day, target]` | Supplied physical target units |
| `pred_raw.npy`, `true_raw.npy` | `[total_windows, pred_len, target]` | Normalized target space |

Here “raw” means before reconstruction/inverse normalization; it does **not**
mean raw physical observations. Window order is basin-major, then chronological
window start. Standard testing reports the complete nominal daily test interval.

Neural observations are reconstructed from float32 normalized batches and
inverse-transformed. Small rounding differences from the original NetCDF are
expected. dHBV reports the loaded physical observations directly and has no
normalized `*_raw.npy` test pair. Missing observations remain NaN.

## Window reconstruction

For prediction length `P`, the implementation takes windows numbered
`0, P, 2P, ...`, concatenates their `P` outputs, and appends any uncovered tail
from the final window. It does not average overlapping forecasts.

```text
P = 3, five windows
window 0: days 0 1 2       ← keep all
window 1: days 1 2 3
window 2: days 2 3 4
window 3: days 3 4 5       ← keep all
window 4: days 4 5 6       ← append day 6
```

For `P=1`, every daily window is used. For larger forecast horizons, the
stitched series mixes lead times. Use raw windows for lead-specific analysis.

## Recompute metrics

```python
from pathlib import Path
import numpy as np
from reignflow.utils.stats.metrics import cal_stations_metrics

run = Path("output/your-run-directory")
pred = np.load(run / "results/pred.npy", allow_pickle=False)
true = np.load(run / "results/true.npy", allow_pickle=False)
metrics = cal_stations_metrics(true[:, :, 0], pred[:, :, 0], ["NSE", "KGE", "Corr", "RMSE"])
print({name: float(np.nanmedian(values)) for name, values in metrics.items()})
```

Each returned metric has one value per basin. Repeat for another target
channel if needed. The final report uses a NaN-aware median across basins,
not a metric calculated after pooling all observations. See [metrics](metrics.md).

## Replay scripts

For an initial training run, `configs_input.sh` appends its checkpoint directory
and `latest` selector. If a source was already supplied, it retains that source.
`configs.sh` records effective settings and points saved training commands to
the new run's own checkpoint, including after a warm start. Review the command
before running it: it is configured for continuation, not automatically for
a new cold-start experiment.

The generated scripts and full provenance can contain machine-local paths.
The [portable records and inference bundle](../guides/evaluate-and-share.md)
are the intended starting points for sharing.
