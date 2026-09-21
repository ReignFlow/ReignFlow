# CLI reference

After installation in a dedicated environment, `python -m reignflow --help`
and `reignflow --help` are available from any working directory. The console
entry point and module call the same application. When using shared runtime
dependencies without installing this checkout, run the module from the repository
root. The examples also assume that root for their relative data and output paths.

The tables below are generated from the real argument parser. They distinguish
**parser defaults** from values later supplied by a dataset profile or resolved
by a model. Run `python scripts/docs_reference.py --check` to detect drift.

## Before choosing options

- `--task_name`, `--model`, and `--data` are required. Implemented tasks are
  `regression` and `forecast`; see [model support](../models/overview.md).
- Default `seq_len` and `pred_len` are both 365. For a last-day neural simulation,
  explicitly set `--pred_len 1`.
- Unset data paths, dates, and variable lists inherit the
  [dataset profile](../data/camels.md#dataset-profiles). `--static_variables None`
  explicitly disables static attributes.
- Most Boolean switches accept an optional value, such as `--do_eval False`.
  `--all_stations` and `--transformer_compile` are presence-only switches.
- Lists use commas without spaces. Quote JSON and paths containing spaces.
- `--patience N` with positive N implies `--save_best`, which implies `--do_eval`.
- Choose at most one of `--resume_from_checkpoint`, `--warm_start_from_checkpoint`,
  and `--inference_bundle`. A bundle requires `--do_test`; warm start is training-only.
- A checkpoint directory requires a selector. An explicit `.pt` file takes
  no selector. Exact resume uses a numbered training-state checkpoint.
- dHBV requires `CompositeRMSE` and `seq_len - warmup_days == pred_len`.
  `nmul` inherits the HBV profile's 24 components when unset.
- `transformer_d_ff` resolves to twice `d_model` when unset. The
  [Transformer replay limitation](../models/transformer.md#checkpoint-limitation)
  still applies even when all CLI arguments parse successfully.

<!-- BEGIN GENERATED: cli -->

## Run and checkpoint options

| Option | Parser default | Meaning and accepted values |
|---|---|---|
| `--help` | `unset` | show this help message and exit |
| `--task_name` | `required` | task name, options: [regression, forecast] |
| `--model` | `required` | model name, options: [LSTM, LSTM_mask, Transformer, dHBV] |
| `--output_dir` | `./output/` | output directory |
| `--des` | `` | exp description |
| `--resume_from_checkpoint` | `unset` | trusted training-state checkpoint used for exact resume, or trusted weights for testing |
| `--warm_start_from_checkpoint` | `unset` | trusted local .pt checkpoint used to initialize model weights for a new training run |
| `--checkpoint_selector` | `unset` | checkpoint selected from a run/checkpoint directory: latest, best, or epoch:N |
| `--inference_bundle` | `unset` | safe public inference bundle directory (requires --do_test) |
| `--export_inference_bundle` | `False` | export selected test weights as a safe public inference bundle |
| `--do_eval` | `False` | whether to do evaluation |
| `--save_best` | `False` | track median basin NSE on the validation set each epoch, save checkpoints/best.pt on improvement, and test on the best weights (implies --do_eval) |
| `--patience` | `0` | early stopping: stop after this many validation epochs without improvement (0 disables; implies --save_best) |
| `--do_test` | `False` | only do test |
| `--seed` | `42` | random seed |

## Data selection

| Option | Parser default | Meaning and accepted values |
|---|---|---|
| `--data` | `required` | dataset name in data folder |
| `--input_nc_file` | `unset` | input nc file |
| `--time_series_variables` | `unset` | time series data names |
| `--target_variables` | `unset` | time series data names |
| `--static_variables` | `unset` | numerical static data names |
| `--train_date_list` | `unset` | train date list |
| `--val_date_list` | `unset` | val date list |
| `--test_date_list` | `unset` | test date list |
| `--station_ids` | `unset` | comma-separated station ids for small selections |
| `--station_file` | `unset` | TXT or CSV station manifest for large selections |
| `--all_stations` | `False` | use every station in source order, overriding the dataset profile |
| `--missing_values` | `unset` | JSON object of raw stored missing sentinels; variable rules override "*" |
| `--gamma_norm_variables` | `unset` | forcing variables for log10(sqrt(x)+0.1) transform; "none" disables |
| `--add_coords` | `False` | whether to add coords |

## Model and window settings

| Option | Parser default | Meaning and accepted values |
|---|---|---|
| `--seq_len` | `365` | input sequence length |
| `--pred_len` | `365` | prediction sequence length |
| `--d_model` | `256` | dimension of model, or hidden size |
| `--dropout` | `0.1` | dropout |
| `--initial_forget_bias` | `3.0` | forget bias for the initial LSTM of decoder in LSTM-based models |
| `--transformer_n_layers` | `4` | number of stacked attention encoder layers |
| `--transformer_n_heads` | `4` | number of attention heads; must divide d_model |
| `--transformer_d_ff` | `unset` | feed-forward width; defaults to twice d_model |
| `--transformer_weight_dropout` | `0.0` | DropConnect rate applied to the Q/K/V projection weights |
| `--transformer_compile` | `False` | torch.compile the encoder to fuse its elementwise kernels |
| `--nmul` | `unset` | number of HBV multi-components (overrides the dataset config Nmul) |
| `--compile_hbv` | `False` | torch.compile the HBV per-step body to cut kernel-launch overhead (same fp32 math; dHBV only) |
| `--warmup_days` | `0` | number of warmup days for dynamic physical models |
| `--export_hbv_parameters` | `True` | export dHBV-generated HBV parameters after long-warm-up testing (default: true) |

## Training and scheduling

| Option | Parser default | Meaning and accepted values |
|---|---|---|
| `--optimizer` | `AdamW` | optimizer |
| `--criterion` | `MaskedMSE` | loss function Choices: `MaskedMSE`, `BasinNormalizedMSE`, `CompositeRMSE`. |
| `--loss_alpha` | `0.25` | CompositeRMSE mix: (1-alpha)*RMSE + alpha*log-space-RMSE |
| `--num_workers` | `0` | data loader workers (0 = load in main process) |
| `--epochs` | `30` | train epochs |
| `--batch_size` | `256` | batch size of train input data |
| `--weight_decay` | `0.0` | optimizer weight decay |
| `--use_amp` | `False` | use CUDA automatic mixed precision for training and dHBV long-warm-up testing |
| `--clip_grad` | `unset` | gradient clipping, None means no clipping |
| `--sampling_strategy` | `all_windows` | training-window sampling: visit every window once or sample with replacement Choices: `all_windows`, `random_windows`. |
| `--learning_rate` | `0.0001` | optimizer learning rate |
| `--scheduler` | `unset` | learning rate scheduler Choices: `LSTMMilestoneLR`, `OneCycleLR`, `CosineAnnealingLR`, `ReduceLROnPlateau`. |
| `--max_lr` | `0.003` | maximum learning rate for OneCycleLR |
| `--pct_start` | `0.3` | percentage of cycle spent increasing learning rate in OneCycleLR |
| `--div_factor` | `10` | initial_lr = max_lr/div_factor for OneCycleLR |
| `--final_div_factor` | `10000.0` | min_lr = max_lr/final_div_factor for OneCycleLR |
| `--anneal_strategy` | `cos` | annealing strategy for OneCycleLR: [cos, linear] |
| `--lr_min` | `unset` | minimum LR for CosineAnnealingLR (default: learning_rate/100) |
| `--swa` | `False` | equally average model parameters over the final training epochs |
| `--swa_last_n` | `10` | number of final epochs included when --swa is enabled |
| `--device` | `auto` | compute device: auto, cpu, cuda:N, or cuda:N,M |
| `--allow_tf32` | `False` | allow TF32 matmul/cudnn (faster on Ampere+ GPUs; tiny precision change) |

## Ignored compatibility options

| Option | Parser default | Meaning and accepted values |
|---|---|---|
| `--physics_model` | `unset` | unsupported physics scaffold option; dHBV uses HBV internally |
| `--param_mode` | `unset` | unsupported physics scaffold option |
| `--num_physics_params` | `unset` | unsupported physics scaffold option |
| `--physics_loss_weight` | `unset` | unsupported physics scaffold option |
| `--param_regularization` | `unset` | unsupported physics scaffold option |
| `--use_residual` | `False` | unsupported physics scaffold option |
| `--dynamic_dropout` | `unset` | unsupported physics scaffold option |
| `--use_physics` | `False` | unsupported physics scaffold option |
| `--use_triton` | `False` | unsupported physics scaffold option |

<!-- END GENERATED: cli -->

## Effective configuration

`update_configs` resolves data defaults and derives `enc_in`, `c_out`, and
`label_len`. These are recorded values, not public CLI arguments. The current
optimizer registry contains `AdamW`, `SGD`, and `Adadelta` even though the
parser does not enumerate them as choices.

The ignored physics options above are accepted for compatibility and cleared
with a warning. They do not enable another physical model. Current executable
examples use the concrete dHBV options instead.

Refer to [training controls](../guides/training-control.md) for loss and
scheduler behavior, [resume](../guides/resume-and-warm-start.md) for continuation,
and [run outputs](outputs.md) for the saved effective configuration.
