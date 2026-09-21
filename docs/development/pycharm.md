# Debug from PyCharm

Open the repository root, select the ReignFlow dependency environment, and use
a **module** run configuration. The root is the directory containing
`pyproject.toml` and `reignflow/`.

| Field | Value |
|---|---|
| Run kind | Module name |
| Module name | `reignflow` |
| Working directory | Repository root |
| Interpreter | Your selected ReignFlow environment |
| Parameters | A complete tutorial command's arguments after `python -m reignflow` |

First run `python -c "import reignflow; print(reignflow.__file__)"` in the
selected interpreter. The path must identify this checkout. A shared
interpreter's bare `reignflow` console script may belong to another installation.

## Use the synthetic example

Create the [quick-start data](../getting-started/quickstart.md#1-create-the-demonstration-data),
then copy the arguments after `python -m reignflow` from the quick-start
training command into the parameters field. Remove Bash line-continuation
backslashes and keep `--device cpu` for the initial session.

## Breakpoints in data-flow order

1. `config_basic.get_config`: parsed values and device choice.
2. `config_basic.update_configs`: profile overrides and derived dimensions.
3. `Preprocessor.subset_split`: ordered stations, periods, and history.
4. `Preprocessor.normalize`: fitted or restored statistics.
5. `Dataset.__getitem__`: input/target alignment and calendar features.
6. Model `forward`: tensor shapes and output space.
7. `NeuralTrainer.compute_loss`: normalized versus physical targets.
8. `NeuralTrainer.test`: reconstruction, inverse transform, and metrics.

For the neural quick start, expect `[B, 14, 3]` forcing, `[B, 2]` attributes,
and `[B, 1, 1]` targets. Transformer additionally requires `[B, 14, 3]` calendar
features. For dHBV, inspect `physics_batch_x` in P/T/PET order and use the
[tested 30/15/15 window](../tutorials/dhbv.md).

## Practical failures

If a cache directory is unwritable, set `MPLCONFIGDIR` and `XDG_CACHE_HOME` to
writable scratch directories. This does not affect the scientific configuration.
Run remote GPU debugging inside a scheduler allocation; CUDA visible IDs remain
relative to that process. See [troubleshooting](../reference/troubleshooting.md)
for data, checkpoint, and device failures.
