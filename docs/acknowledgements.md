# Acknowledgements and code provenance

We thank the [hydroDL](https://github.com/mhpi/hydroDL) and
[NeuralHydrology](https://github.com/neuralhydrology/neuralhydrology) communities
for ideas and insights that informed parts of this work.

The attributions below identify specific components and numerical routines
used or adapted in ReignFlow. We also thank the dPL-HBV,
multiple_forcing, and PyETo authors for making these components available.

The [public release repository](https://github.com/ReignFlow/ReignFlow)
currently contains documentation only. The implementation paths below refer
to the full ReignFlow source checkout described by this documentation.

## Sources of specific components

| Project | Contribution used by ReignFlow | Local implementation |
|---|---|---|
| [hydroDL](https://github.com/mhpi/hydroDL) | CompositeRMSE loss formulation and selected metric conventions | `reignflow/utils/losses/composite_rmse.py` and selected formulas in `reignflow/utils/stats/metrics.py` |
| [dPL-HBV release](https://github.com/mhpi/dPLHBVrelease) | Direct source for the hydroDL-based differentiable HBV implementation, including HBV simulation and gamma routing | `reignflow/models/physics/hbv.py` and `reignflow/models/hybrid/dhbv.py` |
| [NeuralHydrology](https://github.com/neuralhydrology/neuralhydrology) | Selected FLV and FMS calculations, adapted to ReignFlow's NumPy array API | `reignflow/utils/stats/metrics.py` |
| [multiple_forcing](https://github.com/kratzert/multiple_forcing) | Adapted plain LSTM, MSE, and basin-normalized MSE implementations | `reignflow/models/neural/LSTM.py`, `reignflow/utils/losses/MSE.py`, and `reignflow/utils/losses/BasinNormalizedMSE.py` |
| [PyETo](https://github.com/woodcrafty/PyETo) | Potential evapotranspiration utilities | `reignflow/data/pyeto/` |

These attributions identify the reused components. The
[metrics reference](reference/metrics.md#implementation-sources) distinguishes
the NeuralHydrology-derived FLV/FMS calculations and the hydroDL-compatible
metric conventions.

## dHBV provenance and modifications

**ReignFlow's dHBV is modified from the hydroDL-based dPL-HBV implementation**
by Dapeng Feng and collaborators. The direct upstream code reference for the
HBV simulation and routing is
[`mhpi/dPLHBVrelease`](https://github.com/mhpi/dPLHBVrelease), which includes
the hydroDL implementation used for the original dPL-HBV experiments.

ReignFlow adapts that implementation to its batch-first model and data
interfaces, current PyTorch APIs, explicit device handling, fp32 HBV execution,
checkpoint handling, parameter export, and finite-value validation. These are
ReignFlow integration changes; the underlying differentiable parameter
learning approach and upstream HBV implementation retain their original credit.
The [dHBV model page](models/dhbv.md) describes the resulting behavior.

The dPL-HBV source also credits the Beck et al. (2020) NumPy implementation of
HBV-light as an earlier source for its PyTorch HBV model. ReignFlow's direct
source for this component is the dPL-HBV release identified above.

## Scientific references

When reporting experiments, cite the methods and software used as well as the
ReignFlow version. In particular:

- **Hydrologic Transformer:** Liu, J., Bian, Y., Lawson, K., and Shen, C. (2024).
  Probing the limit of hydrologic predictability with the Transformer network.
  *Journal of Hydrology*, 637, 131389.
  [doi:10.1016/j.jhydrol.2024.131389](https://doi.org/10.1016/j.jhydrol.2024.131389).
- **Architecture benchmarking:** Liu, J., et al. (2025).
  From RNNs to Transformers: benchmarking deep learning architectures for
  hydrologic prediction. *Hydrology and Earth System Sciences*, 29, 6811–6828.
  [doi:10.5194/hess-29-6811-2025](https://doi.org/10.5194/hess-29-6811-2025).
- **dPL-HBV:** Feng, D., Liu, J., Lawson, K., and Shen, C. (2022).
  Differentiable, learnable, regionalized process-based models with multiphysical
  outputs can approach state-of-the-art hydrologic prediction accuracy.
  *Water Resources Research*, 58, e2022WR032404.
  [doi:10.1029/2022WR032404](https://doi.org/10.1029/2022WR032404).
- **NeuralHydrology:** Kratzert, F., Gauch, M., Nearing, G., and Klotz, D. (2022).
  NeuralHydrology — A Python library for Deep Learning research in hydrology.
  *Journal of Open Source Software*, 7(71), 4050.
  [doi:10.21105/joss.04050](https://doi.org/10.21105/joss.04050).

The linked upstream repositories provide additional method references and
their software citation instructions. CAMELS users should also cite the
[data sources](data/camels.md).

## Notices and maintenance

The code inventory above summarizes the full source checkout's `THIRD_PARTY_NOTICES.md`.
That inventory records the audited upstream revisions, local scope, and
locations of the complete license texts. Acknowledgements and paper citations
supplement the upstream copyright and license notices; they do not replace
them. Preserve those notices when distributing adapted source files.

When adding or changing an upstream component, update this page, the affected
model or metric page, and the source checkout's notices together. See
[contributing](development/contributing.md#source-and-artifacts).
