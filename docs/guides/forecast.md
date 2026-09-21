# Forecast runoff after the input window

The task determines target alignment:

| Task | Inputs | Targets |
|---|---|---|
| `regression` | `seq_len` forcing days | Last `pred_len` days inside those inputs |
| `forecast` | `seq_len` forcing days | Next `pred_len` days after the inputs |

```text
forecast: [---- observed input forcing ----][--- future targets ---]
                     seq_len                        pred_len
```

Use LSTM, LSTM_mask (CUDA), or Transformer with `0 < pred_len <= seq_len`.
The current adapters return one output per input position; the trainer maps
the final `pred_len` outputs to future targets. There is no autoregressive
decoder, future-weather input, or observed-runoff input unless you explicitly
add an appropriate input variable. The supplied dHBV long-warm-up path is a
regression simulation and is not a supported forecast recipe.

## Run a three-day CPU example

Create the [quick-start data](../getting-started/quickstart.md#1-create-the-demonstration-data)
first, then run:

<!-- example: forecast-train -->
```bash
python -m reignflow --task_name forecast --model LSTM --data CAMELS \
  --input_nc_file output/demo/CAMELS.nc --all_stations \
  --time_series_variables daymet_prcp,daymet_tmean,daymet_pet \
  --static_variables area_gages2,elev_mean \
  --train_date_list 2000-01-01,2000-03-31 \
  --val_date_list 2000-04-01,2000-04-30 \
  --test_date_list 2000-05-01,2000-06-30 \
  --seq_len 14 --pred_len 3 --d_model 16 --dropout 0 \
  --batch_size 32 --epochs 1 --learning_rate 0.001 \
  --do_eval --device cpu --seed 42 --output_dir output/demo/forecast
```

The reader prepends 14 input-history days to each requested split where
available. A candidate sample needs 17 loaded days. The 61-day test period
provides 59 windows per basin, so `pred_raw.npy` has shape `[177, 3, 1]`.

## Understand the saved result

`pred_raw.npy` retains every window and all three lead times, in basin-major,
then window-start order. `pred.npy` has shape `[3, 61, 1]`, but its daily series
is assembled by selecting non-overlapping prediction blocks plus a final tail.
**It is not an average over all overlapping forecasts**, and it is not a
separate score for each lead time.

For lead-specific evaluation, use the raw windows, training scaler, ordered
stations, and dates. The standard four reported metrics describe the stitched
daily series. [Run outputs](../reference/outputs.md#window-reconstruction)
documents the exact selection.

## Replay and limitations

Forecast checkpoints and bundles require the same task, windows, data, and
model settings for evaluation. The CLI still requires target observations and
matching canonical data, so this is a supervised forecast experiment rather
than an operational forecast service. See [evaluate and share](evaluate-and-share.md).

If the file lacks enough history before the nominal test interval, ordinary
testing rejects a reconstructed series of the wrong length. Keep a continuous
daily time axis and supply the requested history.
