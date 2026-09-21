# Contributing

Make changes that a reviewer can connect to a concrete behavior, a test, and
the relevant user documentation.

For this documentation preview, edit the Markdown pages and follow the
[site build instructions](documentation.md#edit-and-verify). The code-related
contribution steps below apply to the full ReignFlow development checkout.

## Work from the intended checkout

Use `python -m reignflow` from the repository root and verify
`reignflow.__file__` when sharing a dependency environment across projects.
Perform package installation checks in a dedicated environment so another
editable checkout is not selected accidentally.

Keep existing uncommitted work separate. A documentation update should describe
current behavior; a numerical or compatibility change needs its own explicit
reason and validation record.

## Validate the affected behavior

For runtime changes, cover the relevant data/model boundary and run the
appropriate [tests and real-data gate](testing.md). For documentation, regenerate
code-derived tables and run the documented command checks and strict build.

```bash
python scripts/docs_reference.py
python -m pytest -q tests/test_docs.py tests/test_documented_workflows.py
python -m mkdocs build --strict
```

Input ordering, missingness, normalization, initialization, sampling, precision,
loss, scheduler timing, checkpoint loading, and metrics can all change numerical
results. Explain the change before updating any benchmark reference. Preserve
historical records and label new results with their source and environment.

## Source and artifacts

Keep generated data, training outputs, checkpoints, caches, and credentials out
of commits. Review portable provenance before sharing it; scientific metadata
remains even when paths are redacted. Load `.pt` files only from trusted sources.

Document new model sources and modifications using the
[integration guide](model-integration.md). Preserve upstream copyright,
attribution, and license files. This preview retains the repository's
[LICENSE](https://github.com/jayhydro/49.ReignFlow_Release/blob/main/LICENSE).
The full code distribution also requires its `THIRD_PARTY_NOTICES.md` and
the accompanying upstream license files.

## Reviewable descriptions

Explain the triggering problem, resulting behavior, tests run, and remaining
limitations. Distinguish a local build from a successful public deployment,
and a small smoke run from a reproduced scientific result.
