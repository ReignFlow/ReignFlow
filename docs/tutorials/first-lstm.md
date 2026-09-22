# Train an LSTM on CAMELS

Complete the [synthetic quick start](../getting-started/quickstart.md) first.
This tutorial moves the same workflow to observed CAMELS data and separates a
small installation check from a scientific comparison.

## Before you start

Run from the full source checkout with the ReignFlow environment active.
Prepare a CAMELS NetCDF containing the variables and basins listed below.

## Prepare the input

Build or locate [CAMELS.nc](../data/camels.md). From the repository root, set
its path in Bash:

```bash
export CAMELS_NC="examples/data_preparation/data/CAMELS_processed/CAMELS.nc"
```

The command below uses three basins, six Daymet forcing variables, and the
profile's 26 numeric attributes. Check that your file includes them. Another
file location is fine; the data identity depends on decoded inputs rather
than its filename.

## Train with disjoint periods

```bash
python -m reignflow --task_name regression --model LSTM --data CAMELS \
  --input_nc_file "$CAMELS_NC" \
  --station_ids 01013500,01022500,01030500 \
  --train_date_list 1999-10-01,2000-09-30 \
  --val_date_list 2000-10-01,2001-09-30 \
  --test_date_list 2001-10-01,2002-09-30 \
  --seq_len 30 --pred_len 1 --d_model 32 \
  --batch_size 64 --epochs 2 --learning_rate 0.001 \
  --save_best --export_inference_bundle \
  --device cpu --seed 42 --des camels-tutorial
```

`seq_len=30` provides 30 forcing days and `pred_len=1` predicts the final day.
Training fits the scaler, including 29 days of preceding history if available.
Validation and test reuse that scaler. `--save_best` enables validation and
selects the highest finite median validation NSE for the final test.

This is a small functional run. Two epochs on three basins establish that
data loading and training work; they do not establish benchmark accuracy.
No particular NSE or wall-clock time is promised.

## Inspect and replay

Keep the run directory printed by training. It holds physical predictions,
observations, four reported metrics, checkpoint weights, resolved configuration,
and provenance. Recompute basin metrics as shown in [outputs](../reference/outputs.md).

For standalone evaluation, repeat the same data/model arguments and add:

```text
--do_test --resume_from_checkpoint path/to/run --checkpoint_selector best
```

Retain the effective validation state: use `--save_best` as above or `--do_eval`.
For the exported bundle, use `--do_test --inference_bundle path/to/run/inference_bundle`
instead of the checkpoint flags. The [quick-start replay](../guides/evaluate-and-share.md#replay-the-quick-start-run)
contains a complete executable example without placeholder arguments.

## Scale the experiment

- Omit the explicit station list to use the profile's 531 basins, or supply a manifest.
- Increase the training period and window length deliberately.
- Choose a loss and scheduler from the [training guide](../guides/training-control.md).
- Use a GPU allocation for longer runs; [Slurm examples](../guides/hpc.md) show the launch pattern.
- Use the exact [benchmark recipe](../reference/benchmarks.md) for comparisons,
  including its forcing products, periods, loss, seed, and precision.

Do not combine a new validation-selected model with a historical last-epoch
benchmark label. Report the selected weight source and normalization version.
