# Maintain and publish the documentation

This preview uses MkDocs Material and contains documentation only. Markdown
lives in `docs/`; `mkdocs.yml` defines navigation and URLs. The preview banner
reminds readers that the public model code release is still being prepared.

## Edit and verify

```bash
python -m pip install -r requirements-docs.txt
python -m mkdocs build --strict
python -m mkdocs serve
```

Use a dedicated documentation environment. The preview does not require a
ReignFlow installation, PyTorch, training data, or a GPU. The local server
normally listens at `http://127.0.0.1:8000/`. Build artifacts go to ignored
`site/`; use `--site-dir` for a scratch destination.

Edit the Markdown pages and run the strict build before publishing an update.
The public preview workflow checks navigation, document targets, and section
anchors. It does not execute training or import the model registry.

The marked CLI, model, and CAMELS profile tables were generated in the full
development checkout with `scripts/docs_reference.py` before export. That script
and the runtime tests are not included here. When changing a default or a
training command, verify it in the development checkout and update the preview
from that verified result. Do not invent new defaults by editing generated tables.

The generator limits model names and registry rows to the documented model
list: LSTM, Transformer, and dHBV. Runtime registration remains separate.

## Validation coverage

The [validation record](../reference/validation.md) identifies the source
snapshot, environment, and results of the latest recorded documentation check.
The matrix below explains what each check establishes.

The following matrix records the cross-checks used in the development checkout
to prepare the documentation. The source files and runtime tests named here
belong to that checkout, not to this documentation-only repository.

| Documentation topic | Code cross-check | Executable evidence |
|---|---|---|
| CLI defaults and choices | `config/config_basic.py` | Generated-table check, help/options tests |
| Documented models and output spaces | `models/registry.py` | Generated registry table, model tests |
| Profile variables, stations, dates | `config/config_dataset_*.py` | Generated profile tables |
| Daily schema, split history, normalization | `data/nc_reader.py`, `data/dataset.py` | Normalization/data tests; physical observation comparison in tutorial runs |
| LSTM train → checkpoint → bundle | `training/neural_trainer.py`, `utils/inference_bundle.py` | Exact Markdown commands and replay array/metric equality |
| Transformer | `models/neural/Transformer.py` | CPU Markdown run; explicit hash-coverage limitation in prose |
| Forecast horizons and reconstruction | `Dataset.__getitem__`, `Preprocessor.restore_data` | Three-day Markdown run, raw-window and physical-date checks |
| dHBV warm-up and routing | `models/hybrid/dhbv.py`, `models/physics/hbv.py` | Routed CPU Markdown run with 30/15/15 window |
| HBV parameter export | `utils/hbv_export.py` | File/dimension checks in the documented dHBV run |
| Model adapter | `ModelEntry` and batch interface | Executable adapter snippet, forward/backward checks |
| Metrics and missingness | `utils/stats/metrics.py` | Metric boundary tests and tutorial metric recomputation |
| Navigation and links | `mkdocs.yml`, all Markdown pages | Strict build with navigation, page, and anchor checks in both checkouts |
| Cluster launch instructions | Slurm template and CLI | Shell syntax and parsed options; no CI job submission |

The synthetic workflows in the development checkout exercise the actual NetCDF
reader and CLI, with no mock trainer or fabricated output log. They do not claim
a CAMELS benchmark or CUDA/AMP validation. The real CAMELS tutorial and
[runtime gates](testing.md#real-data-gate) require the code checkout and a local
dataset. A successful preview build does not rerun or replace those checks.

Current limitations discovered in the review are documented where users need
them: daily-only workflow assumptions, matching-data bundle replay, first-target
validation selection, stitched forecast outputs, the 15-step routed HBV minimum,
and omitted Transformer settings in compatibility hashes.

## GitHub Pages

In repository **Settings → Pages**, select **GitHub Actions** as the publishing
source. The Documentation Preview workflow builds on pull requests, main pushes,
and manual dispatch. Only main-branch non-PR runs in
`ReignFlow/ReignFlow` upload the `site/` artifact and deploy through
the `github-pages` environment. The repository check prevents this preview
workflow from deploying if it is copied to a different repository.
See GitHub's [publishing-source guide](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
and [custom workflow requirements](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages).

The site address is [ReignFlow Documentation Preview](https://reignflow.github.io/ReignFlow/).
After publication, confirm the workflow's deployed URL, homepage, navigation,
preview banner, and search. A local strict build only verifies the artifact;
the successful GitHub Pages deployment establishes that the preview is online.

Public code publication is a separate step. When that release is ready, update
the installation and source links against the released code, rerun its runtime
checks, and remove the preview notice only when the documented workflow is
available to readers.
