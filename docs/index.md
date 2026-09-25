---
hide:
  - navigation
  - toc
---

<div class="rf-hero" markdown="1">

<p class="rf-hero__eyebrow">Documentation preview</p>

# ReignFlow

<p class="rf-hero__lead">Train and evaluate daily rainfall–runoff models from NetCDF data,
with a shared workflow for data preparation, training, and evaluation.</p>

[Get started](getting-started/installation.md){ .md-button .md-button--primary }
[Quick start](getting-started/quickstart.md){ .md-button }

<p class="rf-hero__note">This documentation preview describes the full source checkout; the public
repository currently contains documentation only.</p>

</div>

## Start here

<div class="grid cards rf-grid-2" markdown="1">

-   :material-rocket-launch-outline:{ .lg } __[Getting Started](getting-started/installation.md)__

    Set up the environment and complete your first small CPU run.

-   :material-book-open-variant:{ .lg } __[User Guide](guides/index.md)__

    Prepare data, choose a model, and configure training and evaluation.

-   :material-notebook-outline:{ .lg } __[Examples](tutorials/index.md)__

    Follow complete workflows for CAMELS, Transformer, dHBV, or custom data.

-   :material-file-document-outline:{ .lg } __[Reference](reference/index.md)__

    Look up command-line options, data requirements, metrics, and saved outputs.

</div>

## Models

<div class="grid cards" markdown="1">

-   __[LSTM](models/lstm.md)__

    The first rainfall–runoff baseline for a new dataset.

-   __[Transformer](models/transformer.md)__

    Compare attention with recurrent modeling.

-   __[Differentiable HBV](models/dhbv.md)__

    Learn the parameters of a differentiable HBV simulation.

</div>

See [model selection](models/overview.md) for input requirements and supported
tasks. For model integration, tests, and contributions, visit
[Development](development/index.md).
