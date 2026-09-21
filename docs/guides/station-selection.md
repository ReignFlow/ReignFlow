# Select stations

Choose one selection method. All preserve string IDs and order; leading zeros
are significant.

| Argument | Selection |
|---|---|
| `--station_ids 01013500,01022500` | The listed IDs, in that order |
| `--station_file selections/basins.txt` | A TXT or CSV manifest |
| `--all_stations` | Every source station, in file order |
| None of these | The profile's built-in station list |

These are argument fragments for the [quick start](../getting-started/quickstart.md)
or a [CAMELS run](../tutorials/first-lstm.md). The three explicit options are
mutually exclusive. A custom dataset usually needs a manifest or `--all_stations`
to override the CAMELS list.

## Manifest formats

A TXT file contains one ID per nonblank line:

```text
01013500
01022500
01030500
```

A CSV needs a `station_id` or `station_ids` column:

```csv
station_id,region
01013500,northeast
01022500,northeast
01030500,northeast
```

Other columns are ignored. Preserve the ID column as text in spreadsheet
software. Duplicate, empty, or absent IDs are rejected; the error identifies
the problematic selection.

## Reproducibility and memory

The same ordered IDs selected inline or by manifest give the same model/data
compatibility identity. Manifest/all-source provenance records the station
count and an ordered digest instead of expanding a potentially large list.
Keep the actual manifest with your experiment: a digest cannot reconstruct it.

This equivalence concerns data selection and the model/data hash. Exact
training resume also checks its separate training-settings contract, so retain
the original selection mode when continuing an interrupted run.

Data loading eagerly materializes selected arrays for each split. A large
manifest can be parsed without implying that the resulting time-series arrays
will fit in memory. Start with a small representative subset and measure memory
before scaling.
