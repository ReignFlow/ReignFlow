# Examples

Choose a workflow and follow it from input data to saved results. Run the
commands from the full source checkout after [installation](../getting-started/installation.md).
The [quick start](../getting-started/quickstart.md) is the shortest first run.

| Example | Input | What you will do |
|---|---|---|
| [LSTM on CAMELS](first-lstm.md) | A prepared CAMELS NetCDF | Train on observed data, select weights with validation, and inspect results |
| [Transformer](transformer.md) | Small synthetic NetCDF | Run an attention model with explicit window and architecture settings |
| [Differentiable HBV](dhbv.md) | Small synthetic NetCDF with P/T/PET | Train with physical forcing and inspect exported HBV parameters |
| [Custom daily data](custom-data.md) | A NetCDF created in the example | Define your own variable names and run the full training command |

Each example identifies its inputs, steps, and outputs. Small synthetic runs
check the workflow; [benchmark recipes](../reference/benchmarks.md) describe
scientific experiments with their own data and settings.

After a first run, follow [checkpoint and bundle replay](../guides/evaluate-and-share.md#replay-the-quick-start-run)
or the [three-day forecast example](../guides/forecast.md#run-a-three-day-cpu-example).
Use the [user guide](../guides/index.md) to understand a setting and the
[reference](../reference/index.md) to look up its exact definition.
