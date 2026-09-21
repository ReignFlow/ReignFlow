# Validation and model selection

The default final test uses the last training weights. Validation and weight
averaging change that selection as follows.

| Flag | Effect | Implies |
|---|---|---|
| `--do_eval` | Compute validation loss and median basin NSE each epoch | — |
| `--save_best` | Save improved validation weights to `best.pt` | `--do_eval` |
| `--patience N` | Stop after N validation epochs without improvement | `--save_best` when N > 0 |
| `--swa` | Average weights from the final planned epochs | — |

Use validation dates disjoint from training and testing. The framework does
not enforce this separation for you. For several target channels, validation
selection uses NSE of **the first target**; the final test reports all targets.

## Which weights are tested

| Training options | Final test weights |
|---|---|
| No selection, or `--do_eval` alone | Last epoch |
| `--save_best` | Best finite median validation NSE; last epoch if no finite best exists |
| `--swa` alone | Average when snapshots exist; otherwise last epoch |
| `--save_best --swa` | Higher finite validation NSE of best and average; best wins a tie |

When both selection methods lack a usable finite candidate, last-epoch weights
are retained. In standalone `--do_test`, the explicit checkpoint or bundle is
already the selected source; these training flags do not pick a different file.

## Validation behavior

Validation runs in fp32. It reconstructs window predictions and evaluates
physical-space NSE per basin, then takes the NaN-aware median. This includes
the standard metric mask, which removes negative observations by default.
The validation loss is an average over usable batches and need not rank epochs
in the same order as median NSE.

dHBV validation uses its training-style internal warm-up windows. Its final
test instead parameterizes each basin from the full training period and uses
long-warm-up simulation. These are different evaluation paths.

## Early stopping

Positive `--patience` counts epochs without a validation NSE improvement.
The counter, best value, and best epoch are restored by exact resume.
`ReduceLROnPlateau` has its own fixed patience of five and monitors loss;
see [scheduling](training-control.md#learning-rate-schedules).

## Weight averaging

`--swa_last_n` defaults to 10. The averaging window starts at
`max(1, epochs - swa_last_n + 1)` and averages equally under the existing
optimizer and scheduler. There is no separate SWALR schedule.

Early stopping may leave fewer snapshots, or none if it stops before the
window. A completed average is stored in `checkpoints/swa.pt`; the in-progress
average and its history are part of the numbered exact-resume checkpoint.

To evaluate an average later, supply the explicit `checkpoints/swa.pt` path.
Directory selectors offer `latest`, `best`, and `epoch:N` only. The
[checkpoint reference](checkpoints.md) explains the file types.
