# Checkpoints and provenance

This is the file and compatibility reference. Follow
[resume and warm start](resume-and-warm-start.md) or
[evaluate and share](evaluate-and-share.md) for task instructions.

## Checkpoint types

| File | Contents | Use |
|---|---|---|
| `checkpoint_epoch_N.pt` | Completed epoch, model, optimizer, scheduler, fitted scaler, RNG, AMP/SWA state, histories, selection state, hashes | Exact resume, warm start, test |
| `best.pt` | Best-validation model, scaler and compatibility information | Warm start or test |
| `swa.pt` | Averaged model, averaging/selection metadata, scaler and compatibility information | Warm start or test |
| `inference_bundle/` | Safetensors weights, numeric scaler, portable metadata, station list | Matching-data test only |

The current checkpoint format is version 6; the inference-bundle schema is
version 2. Format version alone is insufficient for compatibility.

Numbered and selected checkpoints use a temporary file followed by atomic
replace. After a successful numbered save, the immediately preceding epoch is
removed. Failed publication leaves the previous checkpoint intact; a cleanup
failure can leave multiple complete numbered files. A failure before the first
successful checkpoint cannot be resumed.

## Verification hashes

| Record | Protects |
|---|---|
| Model/data contract hash | Selected model settings, profile/data semantics, normalization |
| Canonical model-input fingerprint | Ordered stations/dates and decoded raw/normalized arrays for loaded splits |
| Exact-resume training hash | Enumerated optimizer, schedule, sampling, precision, validation, and training settings |

Canonical array hashing separates values and NaN masks. Storage-only changes
such as NetCDF compression do not matter when decoded values and metadata
semantics are equivalent. Changing numerical values while converting dtype
can still change the fingerprint.

Validation contributes to the input fingerprint when loaded. This is why a
test command must retain the original effective `--do_eval` state. Selection
through equivalent ordered inline IDs or manifests shares a model/data identity;
exact resume has an additional training-settings check.

Hashes cover explicitly enumerated fields, not every future CLI argument.
The current Transformer-specific options are recorded but omitted from those
field lists. Compare them manually as described on the
[Transformer page](../models/transformer.md#checkpoint-limitation).
Source revision and environment are recorded, but they are not universal
runtime compatibility enforcement.

## Normalization compatibility

The forward transform and its inverse now share epsilon:

```text
z = (q - mean) / (std + 1e-5)
q = z * (std + 1e-5) + mean
```

The earlier inverse used `z * std + mean`. Its archived physical arrays and
metrics retain that historical convention. The changed normalization contract
rejects ordinary checkpoint/bundle replay and exact resume from those runs,
even if their format version is still 6. The dHBV physical-output path does
not use this inverse, but its metadata can still carry the changed contract.

To continue research, choose a new corrected run or a **model-only warm start**
from a trusted same-format, shape-compatible checkpoint. Warm start refits the
scaler and starts new training state; it does not reproduce the old experiment.
Keep historical artifacts and corrected evaluations separately labeled. Do not
edit hashes merely to suppress a compatibility error.

## Run-level records

- `provenance.json`: full effective configuration, dataset settings, source
  state, normalization, sampling, and fingerprints.
- `provenance_portable.json`: a copy with recognized local paths redacted;
  scientific values remain.
- `configs.sh`: resolved effective command. Saved training commands point at
  the run's own latest checkpoint for continuation.
- `configs_input.sh`: original arguments, with resume settings appended for
  an initial training invocation.

Checkpoints carry fitted statistics and hash references rather than complete
configuration text. Retain the sidecars and original data/selection records
with the checkpoint. Review records before sharing; a portable path policy
does not remove all scientific metadata.

The `.pt` loader enables pickle because training state includes Python objects.
Use it only for trusted files. Public evaluation bundles use safetensors and
`np.load(..., allow_pickle=False)`.
