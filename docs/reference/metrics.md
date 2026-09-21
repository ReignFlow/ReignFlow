# Hydrological metrics

The CLI reports **NSE, KGE, Corr, and RMSE** separately for each target. Each
metric is calculated per basin and summarized with `numpy.nanmedian` across
basins. This differs from pooling all basin observations into one time series.

## Python API

```python
from reignflow.utils.stats.metrics import cal_stations_metrics

metrics = cal_stations_metrics(
    observations, simulations,  # same shape: [basin, time]
    ["NSE", "KGE", "Corr", "RMSE"],
)
```

The return value maps names to one-dimensional per-basin arrays. With no
`metrics_list`, the API defaults to NSE, KGE, RMSE, Corr, FHV, and FLV; that
API default is broader than the four metrics printed by the CLI. There is no
CLI flag for choosing another metric list in this checkout.

## Masks and undefined values

Paired metrics use finite observations and simulations. `remove_neg=True`
(the default) additionally excludes negative observations. Negative predictions
are retained; the neural models do not automatically clip runoff.

Undefined calculations return NaN, including NSE with constant observations,
correlation with a constant series, or required zero denominators. Metrics such
as NSE/KGE require at least two paired observations. Report how many basins
have finite scores when interpreting a median.

`logNSE` uses only strictly positive pairs. `ubRMSE` uses independent finite
means before pairing anomalies and intentionally ignores `remove_neg`.
Retained negative observations make FLV/FMS undefined; those FDC calculations
replace nonpositive predictions and zero observations with a small value
inside their logarithms.

## Definitions

In this table `o` is observation, `s` simulation, `r` Pearson correlation,
and means/standard deviations use the relevant valid samples. Standard
deviations are population values.

| Name | Implemented definition or convention |
|---|---|
| `NSE` | `1 - sum((s-o)^2) / sum((o-mean(o))^2)` |
| `R2` | Alias of NSE; not squared correlation |
| `Corr` | Pearson correlation |
| `PearsonR2` | Squared Pearson correlation |
| `CorrSp` | Spearman rank correlation |
| `KGE` | `1 - sqrt((r-1)^2 + (std(s)/std(o)-1)^2 + (mean(s)/mean(o)-1)^2)` |
| `KGE12` | KGE with the coefficient-of-variation ratio replacing the standard-deviation ratio |
| `RMSE`, `MSE` | Square root of mean squared error; mean squared error |
| `MAE` | Mean absolute error |
| `Bias` | `mean(s-o)` |
| `PBias` | `100 * sum(s-o) / sum(o)`; positive is overestimation |
| `ubRMSE` | RMSE of anomalies around each series' independent mean |
| `Alpha-NSE` | `std(s) / std(o)` |
| `Beta-NSE` | `(mean(s)-mean(o)) / std(o)` |
| `Beta-KGE` | `mean(s) / mean(o)` |
| `FHV` | High-flow bias using the ascending FDC from `round(0.98*n)` onward |
| `FHV_k` | Same high-flow rule for integer percentage `0 < k < 100` |
| `FLV` | Log-FDC low-volume bias using the rounded lowest 30% selection |
| `FMS` | Relative log-FDC slope difference at rounded 20%/70% positions |
| `logNSE` | NSE after natural logarithms of positive paired values |
| `NSEanomaly` | NSE after independently centering the paired series by their full-period means |

RMSE, MAE, and Bias retain the target units; MSE has squared units. NSE/KGE
can be negative and have optimum 1. FDC percentage conventions and rounded
selection boundaries matter, especially for short samples. Do not substitute
another library's FHV or FLV formula without checking equivalence.

## Normalization and comparisons

Standard reporting uses physical arrays. Neural observations have passed
through normalization and float32 batch tensors before reconstruction;
rounding near zero can affect the default negative-observation mask. dHBV
uses loaded raw physical observations. Recompute comparisons using the same
mask and inverse convention rather than assuming a normalization correction
leaves every metric unchanged.

The formula and boundary tests are in `tests/test_metrics.py`; calculation
lives in `reignflow/utils/stats/metrics.py`. [Run outputs](outputs.md#recompute-metrics)
shows how to reproduce a report from saved arrays.
