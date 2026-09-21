# Resume or warm start

Use exact resume to continue an interrupted experiment. Use warm start when
the training plan or data changes but the model weights remain compatible.

| Operation | Restores | Run directory |
|---|---|---|
| Exact resume | Model, optimizer, scheduler, scaler, RNG, AMP, SWA, histories, early-stopping state | Original run |
| Warm start | Model tensors only | New run |

Both load trusted `.pt` files. For shared evaluation weights, use an
[inference bundle](evaluate-and-share.md).

## Exact resume

Repeat the interrupted run's settings and add these argument fragments:

```text
--resume_from_checkpoint path/to/run --checkpoint_selector latest
```

Do not add `--do_test`. The next epoch follows the last saved completed epoch;
there is no mid-batch recovery. A numbered checkpoint contains the full
training state and is written atomically. The previous epoch is removed only
after the new checkpoint is published.

The saved training `configs.sh` already points to the run's own latest
checkpoint. After checking its environment and data paths, it can be run with:

```bash
bash path/to/run/configs.sh
```

The exact-resume contract includes the requested total `--epochs`. Increasing
it is a new training plan, not exact continuation. Keep the original optimizer,
schedule, sampler, batch size, validation state, data, and precision settings.
Missing state or mismatched contracts fail instead of silently resetting state.

CPU/GPU execution class and active CUDA RNG-state count must agree. Changing
visible GPU numbering is different from changing the number of GPUs. Even
compatible contracts do not guarantee bitwise behavior on different hardware.

## Select a checkpoint

| Source | Selector |
|---|---|
| Run/checkpoint directory | Explicit `latest`, `best`, or `epoch:N` |
| Explicit `.pt` file | No selector |
| `swa.pt` | Explicit file path |

`latest` means the highest existing numbered checkpoint. `best` never falls
back to latest. Most normal runs retain only their newest numbered epoch;
`epoch:N` cannot recover a deleted earlier file.

Exact resume requires `checkpoint_kind=training_state`. `best.pt` and `swa.pt`
are selected weights for testing or warm start, not optimizer continuation.

## Warm start

Use your new training command with:

```text
--warm_start_from_checkpoint path/to/run --checkpoint_selector best
```

The new run fits a new scaler, starts at epoch 1, and initializes a fresh
optimizer, scheduler, random state, AMP state, SWA history, and early-stopping
counter. Dates, stations, learning rate, and epoch count may change when the
architecture remains compatible.

State-dictionary loading is strict. Changing the hidden size, number of inputs,
targets, or HBV components usually changes tensor shapes and prevents loading.
Shape compatibility alone does not make changed input order or units a sensible
transfer; review their scientific meaning.

The loader accepts the current checkpoint format. A same-format checkpoint
using the former inverse-normalization contract can initialize weights through
warm start, which starts a new experiment; it cannot become an exact replay by
changing a hash. See [normalization compatibility](checkpoints.md#normalization-compatibility).

## Transformer limitation

The current hash field lists omit Transformer-specific options. Preserve and
compare all `transformer_*` settings manually when resuming, even if the
reported contracts match. Some changes alter behavior without altering tensor
shapes. [The model page](../models/transformer.md#checkpoint-limitation) records
the exact limitation.
