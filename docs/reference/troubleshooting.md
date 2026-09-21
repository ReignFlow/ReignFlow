# Troubleshooting

Match the failure to the stage below. Exact paths, hashes, and array sizes vary;
the quoted phrases identify errors in the current code.

## Data loading

| Symptom | Check and remedy |
|---|---|
| NetCDF file not found | Set `--input_nc_file`; the profile default is relative to the source checkout |
| Station ID absent | Preserve leading zeros; choose IDs in the file or use `--all_stations` |
| Selected 0 dates | Check inclusive split dates against the source time range |
| Missing forcing/attribute | CLI variable lists replace profile lists; override all incompatible inherited names |
| No forcing/target coverage | Inspect NaNs and declared sentinels; a saved scaler does not bypass coverage checks |
| Infinite normalized values | Check raw values, declared missing values, and fitted/restored statistics |

### Split too short for one window

The `split is too short to create sliding windows` message reports loaded days
and `sample_len`. Neural regression needs `seq_len` loaded days; forecast needs
`seq_len + pred_len`. Earlier history may be prepended. dHBV uses internal
warm-up and needs a full window inside each constructed split.

### Incomplete test history

`Restored test prediction length does not match the nominal test date range`
means reconstructed output does not cover the requested daily interval. Supply
more preceding history, move the test start, or shorten the input window.
Check for gaps/duplicates in the daily axis as well.

### Units metadata warning

The dHBV message `using the profile-declared units for validation` means the
file omitted a unit attribute. It is not a units conversion. Confirm the data's
physical units, then regenerate or correct metadata when appropriate.
Incompatible declared units fail explicitly.

### Suspicious sentinels

Undeclared extreme values trigger `DATA QUALITY WARNING` and remain unchanged.
If they truly represent missing data, declare raw sentinels with
`--missing_values` or repair NetCDF metadata. Do not automatically classify
every negative or zero value as missing.

## dHBV

| Error or symptom | Remedy |
|---|---|
| `seq_len - warmup_days == pred_len` | Choose consistent window lengths |
| A short example fails in `UH_gamma` with a 15-step shape | The routed profile needs `pred_len >= 15`; use the [tested example](../tutorials/dhbv.md) |
| Test must be one day after training ends | Make test contiguous with train; this is checked after training |
| `requires --criterion CompositeRMSE` | Select the physical-space loss |
| Nonfinite physical output | Inspect raw P/T/PET; normalized zero filling does not impute physical training forcing |
| Random-window coverage is at least 1 | Reduce batch size, use more basin/time data, or use `all_windows` |

Only the regression long-warm-up simulation is documented for dHBV. Parsing a
forecast flag does not establish a valid physical forecast workflow.

## Checkpoint refuses to load

### Model-input fingerprint mismatch

The decoded inputs differ from the source run. Compare dates, ordered
stations/variables, windows, transforms, missing-value rules, and the effective
validation state. A run trained with validation needs that split loaded during
evaluation too. Review `configs.sh`.

### Model/data contract mismatch

Check model settings and normalization version. The old inverse-normalization
contract is intentionally incompatible. See
[normalization compatibility](../guides/checkpoints.md#normalization-compatibility).

### Exact-resume training contract mismatch

Exact resume continues the same plan, including `--epochs`, optimizer, schedule,
batching, sampler, and precision. To extend or change the plan, use a new
[warm start](../guides/resume-and-warm-start.md).

### Selector or source errors

Directories require `--checkpoint_selector latest`, `best`, or `epoch:N`.
Explicit `.pt` files take no selector. `swa.pt` is an explicit path, and selected
best/SWA weights are not full exact-resume state. `--do_test` needs a checkpoint
or inference bundle; the program does not choose an arbitrary local run.

### State-dictionary mismatch

Warm start still loads tensors strictly. Changed input/hidden/output dimensions,
HBV component count, or Transformer compilation can change tensor keys/shapes.
Use the matching architecture or start fresh. A same-shaped Transformer can
also change behavior without a hash mismatch, so compare every Transformer
option manually.

### Bundle rejects new dates or stations

The bundle interface replays matching canonical inputs. New periods, station
order, values, or variables require a different workflow; bundles are not
currently a general transfer or observation-free inference format.

## Device and training

`LSTM_mask` requires CUDA. Other documented models support small CPU runs.
Explicit CUDA IDs are local to the process's visible devices; get a scheduler
allocation first. ReignFlow does not reset `CUDA_VISIBLE_DEVICES`.

Multiple IDs use DataParallel on one node, not multi-node training. The dHBV
long-warm-up test uses its unwrapped primary-device core. See [cluster jobs](../guides/hpc.md).

`ReduceLROnPlateau requires --do_eval` means validation must be enabled.
`--save_best` and positive `--patience` enable it implicitly. Nonfinite targets,
outputs, losses, or gradients require data/numerical inspection; a finite loss
alone is not evidence of useful hydrological skill.

When reporting a failure, include the command, environment, and console error.
`results.txt` is not a full console capture; scheduler stdout/stderr or a captured
terminal log may contain the traceback. Remove local paths you do not intend
to share.
