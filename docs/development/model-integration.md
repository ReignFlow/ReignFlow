# Add a model

Keep a literature implementation close to its audited source and place the
ReignFlow interface in a small adapter. Models should receive tensors, return
predictions, and leave data selection, normalization, device placement, loss,
and file output to the existing framework.

## Stable model contract

`Model(configs, config_dataset)` is a PyTorch module. Its `forward` accepts
the [batch dictionary](../models/overview.md#inputs-and-outputs) and returns
`outputs_time_series` as a finite tensor `[B, T_out, Y]`. The trainer scores
the final `pred_len` steps, so provide at least that many outputs.

Dynamic inputs are `[B, T, F]`; static inputs are `[B, C]`, including a possible
zero-width attribute axis. `batch_x_time_stamp` supplies `[B, T, 3]` calendar
features. Raw targets and optional `physics_batch_x` have separate roles;
there is no general `raw_batch_x` in the standard batch.

The following complete adapter can be placed in
`reignflow/models/neural/paper_model.py`. It illustrates the interface with a
linear projection; replace that core with the actual model.

<!-- example: model-adapter -->
```python
import torch.nn as nn
from reignflow.utils.validation_tools import combine_timeseries_and_statics


class Model(nn.Module):
    def __init__(self, configs, config_dataset):
        super().__init__()
        self.projection = nn.Linear(configs.enc_in, configs.c_out)

    def forward(self, batch_data_dict):
        inputs = combine_timeseries_and_statics(
            batch_data_dict["batch_x"], batch_data_dict["batch_c"]
        )
        prediction = self.projection(inputs)
        return {"outputs_time_series": prediction}
```

## Register behavior explicitly

The actual registry entry has four fields:

```python
from dataclasses import dataclass
from typing import Optional


@dataclass(frozen=True)
class ModelEntry:
    factory: type
    prediction_space: str = "normalized"
    inference_mode: str = "window"
    required_criterion: Optional[str] = None
```

Add your factory to `MODEL_REGISTRY` and update the CLI model help.
`prediction_space` controls training targets and metric conversion;
`inference_mode` controls final testing. The current physical entry is dHBV,
using `physical`, `dhbv_long_warmup`, and required `CompositeRMSE`.
Defining a new string does not implement a new trainer path.

Add model-specific CLI options with explicit names. Include behavior-affecting
settings in the compatibility and exact-resume field lists in `provenance.py`,
not only in the saved configuration. The present
[Transformer omission](../models/transformer.md#checkpoint-limitation)
shows why recording an option and enforcing it are different.

## Preserve upstream attribution

For copied multi-file code, keep `core.py` and `adapter.py` separate where that
helps review. Include `SOURCE.md` with the paper, upstream repository, exact
revision, copied files, and local changes; retain the complete `LICENSE.txt`
and update `THIRD_PARTY_NOTICES.md`. Do not replace upstream authorship with
the repository's maintainer identity.

## Validate the integration

1. Compare a fixed upstream forward result before changing interfaces.
2. Check batch shapes, absent statics, calendar/physics requirements, finite
   outputs, gradients, and device movement.
3. Check state-dictionary and inference-bundle round trips.
4. Verify every scientific option is recorded and compatibility-checked.
5. Run a 20-basin real-data trial with the model's intended data/loss/test path.
6. Add/update the relevant [gate](testing.md#real-data-gate); the existing gate
   does not automatically exercise a newly registered model.
7. Regenerate documentation tables and run documentation checks.

No automatic plugin discovery is used. An explicit small registry remains the
integration point; introduce new abstractions only when an implemented model
needs them. Name concrete physical models independently, with their own
forcing, unit, loss, and inference contracts.
