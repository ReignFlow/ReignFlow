# Testing

Use checks appropriate to the change. Documentation examples, synthetic
integration tests, real-data checks, and GPU reproduction establish different
things; a skipped test does not count as a successful integration.

## Core tests

```bash
python -m pytest -q
python -m compileall -q reignflow tests scripts
```

The suite covers data selection and missingness, normalization, models,
losses/metrics, sampling, scheduling, checkpoint/resume, bundles, HBV exports,
and launchers. CUDA-dependent cases may skip on a CPU runner. Parser stress
tests for large station manifests do not load corresponding time-series data.

## Documentation checks

```bash
python -m pip install pytest -r requirements-docs.txt
python scripts/docs_reference.py --check
python -m pytest -q tests/test_docs.py tests/test_documented_workflows.py
python -m mkdocs build --strict
```

These verify generated tables against the parser/registries, links and assets,
CLI help, shell syntax/options, and the executable model-adapter example. The
workflow tests run the Markdown commands on a temporary synthetic NetCDF:
LSTM training/checkpoint/bundle replay, Transformer, three-day forecasting,
and dHBV with parameter export. Physical observations are compared with the
input NetCDF, not only with another normalized result.

The tests write experiment artifacts to temporary directories. A strict site
build additionally checks navigation, document targets, and section anchors.
[Documentation maintenance](documentation.md) describes generation and publishing.

## Real-data gate

The existing gate uses `tests/fixtures/camels_20_stations.txt` and a local
CAMELS file. The documented CPU suite runs LSTM and dHBV.
This CPU check does not cover Transformer or validate CUDA execution. Run it
when a runtime change affects data, training, normalization, checkpointing, or metrics.

```bash
export REIGNFLOW_CAMELS_NC="path/to/CAMELS.nc"
python scripts/run_real_camels_gate.py --suite cpu --require-real-data
```

`--require-real-data` makes unavailable required data an error. Without it,
missing prerequisites can produce a skipped or partial report. The default
output is `output/real_camels_gate/`; override it with the gate's
`--output-dir` option.

The gate checks station order, update counts, shapes, finite outputs, metrics,
and expected hashes/tolerances, including HBV exports. Read the JSON report
and compare numerical changes with the approved reference; do not adjust
expected hashes just to make a changed algorithm pass.

## Release and scientific claims

Public CI does not include the large CAMELS file or guarantee CUDA availability.
Documentation-only changes need not retrain production models. CPU tutorial
success does not prove GPU precision behavior, long-run convergence, or an
entire benchmark matrix. Archive a separate record for those claims.

Keep scientific datasets and run artifacts unchanged while reviewing prose.
When a documented limitation exposes a runtime issue, record it explicitly
and scope the runtime change separately from editorial corrections.
