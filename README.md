# ReignFlow documentation preview

This repository publishes the ReignFlow documentation for review before the
public code release. It contains Markdown, illustrations, styling, and the
MkDocs build configuration. Model implementations, training scripts, scientific
datasets, checkpoints, and the development repository's Git history are not
included.

Read the [documentation preview](https://reignflow.github.io/ReignFlow/).

## Preview locally

Use Python 3.12 and a dedicated documentation environment:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements-docs.txt
python -m mkdocs build --strict
python -m mkdocs serve
```

Open `http://127.0.0.1:8000/`. No ReignFlow installation, PyTorch, NetCDF reader,
training data, or GPU is required to build this site.

## Review and edit

Edit `docs/` for content, `mkdocs.yml` for navigation, and
`docs/stylesheets/extra.css` for styling. Runtime examples describe the full
ReignFlow source checkout; they cannot be executed from this documentation-only
repository. Changes to those commands should be checked in the development
checkout before updating the corresponding text here.

The [documentation maintenance page](docs/development/documentation.md) explains
the distinction between a site build and the earlier code cross-checks.

## Publish the preview

In GitHub repository **Settings → Pages**, choose **GitHub Actions**. The
Documentation Preview workflow builds pull requests and publishes successful
main-branch builds only from `ReignFlow/ReignFlow`.

Keep code publication as a separate change. Inspect the files before the first
push: a branch in a public repository is already visible before it is merged.

## Acknowledgements

ReignFlow is an independently designed rainfall–runoff modeling framework.
We thank [hydroDL](https://github.com/mhpi/hydroDL) and
[NeuralHydrology](https://github.com/neuralhydrology/neuralhydrology) for ideas
and insights that informed parts of this work.
See [acknowledgements](docs/acknowledgements.md) for related projects and
scientific references.

## License

The documentation snapshot retains the original repository's [LICENSE](LICENSE).
