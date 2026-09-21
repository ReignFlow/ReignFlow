# Validation record

**Checked on 21 September 2026.** This record describes the development
snapshot used to check these documentation examples. The public repository
currently contains documentation only; this is not an installable code release.

## What was checked

| Check | Result |
|---|---|
| Exact commands from the Markdown examples | 5 CPU workflow tests passed: LSTM, Transformer, three-day forecast, dHBV, and custom data |
| Checkpoint and inference-bundle replay | Predictions, observations, NSE, and RMSE exactly matched the original LSTM run |
| Output interpretation | Physical observations matched the input NetCDF on the documented dates; predictions and metrics were finite |
| Custom data and HBV export | String station IDs retained leading zeros; saved arrays and HBV parameter dimensions matched the examples |
| Documentation, CLI, and adapter checks | 15 tests passed; generated references matched the parser, model registry, and dataset profiles |
| Strict website builds | Development and public-preview documentation both passed; optional tables/code and the result image rendered |

The [quick-start figure](../getting-started/quickstart.md#3-inspect-the-predictions)
comes from this run: prediction shape `[3, 61, 1]`, median NSE **−0.1450**,
and median RMSE **0.7045 mm/day**. These are two-epoch results on artificial
data, useful for checking the workflow rather than judging model accuracy.

## Environment and source

| Component | Tested value |
|---|---|
| Python / PyTorch | 3.12.14 / 2.7.1+cu118; execution on CPU |
| NumPy / xarray / netCDF4 | 2.5.2 / 2026.7.0 / 1.7.4 |
| pytest | 9.1.1 |
| MkDocs / Material | 1.6.1 / 9.7.6 |

The runtime checkout was based on commit
`a4987aa92fb131d3bbbf030c13b063e7fe507148` and contained uncommitted changes.
That commit alone therefore does not identify the tested code. The
[machine-readable summary](../assets/validation-summary.json) records a SHA-256
fingerprint of the 57 runtime/configuration files in its stated scope, the
environment, and check outcomes. It identifies this snapshot without publishing
the development source.

The workflow tests emitted one NumPy binary-interface warning while importing
a compiled dependency. The checks above passed; this record does not establish
compatibility for every dependency combination.

## Limits

This review did not rerun GPU, AMP, compilation, or full CAMELS benchmark
experiments. [Historical benchmark results](benchmarks.md) have their own data
and experiment settings. Passing these small CPU examples does not validate
those scores or make different benchmark rows directly comparable.

When runtime defaults or tutorial commands change, rerun the checks and update
this record. The public Pages workflow builds the website; it does not execute
training. See [documentation maintenance](../development/documentation.md)
for the check coverage and publishing process.
