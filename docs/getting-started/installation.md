# Installation

!!! info "Public code release pending"

    This preview repository contains documentation only. The commands below
    require a full ReignFlow source checkout; cloning the documentation repository
    does not provide the Python package. Public code download instructions will
    be added with the code release.

Use Python 3.12 for the documented workflow. The package declares Python 3.9
or newer, but that declaration does not establish that every supported Python,
PyTorch, and CUDA combination has been tested.

## Create an environment

```bash
conda create -n reignflow-env python=3.12 -y
conda activate reignflow-env
```

Alternatively, create a virtual environment with `python3.12 -m venv .venv`
and activate it with `source .venv/bin/activate`.

## Install from a full source checkout

Use the dedicated environment created above:

```bash
python -m pip install -e .
python -m reignflow --help
```

The repository root is the directory containing `pyproject.toml`, `mkdocs.yml`,
and the `reignflow/` package. Run the documentation commands there.

Runtime dependencies include PyTorch, NumPy, pandas, xarray, NetCDF4, SciPy,
matplotlib, tqdm, and safetensors. Choose a PyTorch build compatible with your
machine before installing the project if your cluster provides a particular
CUDA environment. Optional basin maps use `python -m pip install -e '.[viz]'`.

## Check the interpreter and device

```bash
python -c "import reignflow, torch; print(reignflow.__file__); print(torch.__version__); print(torch.cuda.is_available())"
```

The package path should identify this checkout. LSTM, Transformer, and dHBV
support CPU execution; LSTM_mask requires CUDA. On a cluster, check CUDA from
inside a [GPU allocation](../guides/hpc.md).

<details markdown="1">
<summary>Several ReignFlow checkouts share an environment</summary>

When several projects share the same interpreter and package name, run
`python -m reignflow` from this checkout with the dependencies already installed.
Avoid installing several editable ReignFlow projects into the same environment:
the console command and imports can otherwise resolve to another checkout.

</details>

## Development dependencies

In the full source checkout:

```bash
python -m pip install pytest -r requirements-docs.txt
python -m pytest -q tests/test_docs.py tests/test_documented_workflows.py
python -m mkdocs build --strict
```

These are needed for code development and documentation validation. To build
this documentation-only preview, use the separate
[documentation build instructions](../development/documentation.md#edit-and-verify).
The [quick start](quickstart.md) describes the model workflow for the code release.
