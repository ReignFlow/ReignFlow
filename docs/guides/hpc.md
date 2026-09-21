# GPU jobs on Slurm

Use a compute allocation for GPU training. Activate the same environment on
the compute node and run from the repository root. GPU IDs in ReignFlow are
relative to the devices exposed by the scheduler.

## Interactive check at TTU

A TTU example is:

```bash
interactive -c 4 -g 1 -p matador -t 00:30:00
conda activate reignflow-env
python -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.device_count())"
```

Run the training command inside the allocated shell using `--device cuda:0`.
Partition availability, environment initialization, and CUDA modules are site
settings; consult the [TTU environment guide](https://www.depts.ttu.edu/hpcc/userguides/general_guides/environment-update-2025.php).
An interactive allocation does not install dependencies or select a compatible
PyTorch build for you.

## Independent experiments

A Slurm array gives each experiment its own allocation. The following
template runs three small CAMELS seeds. Save it as `train-array.sbatch` in
the repository root and adjust the site resource requests and data path.
The Bash shell must already be able to initialize your Conda installation.

```bash
#!/bin/bash
#SBATCH --job-name=reignflow
#SBATCH --partition=matador
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --gres=gpu:1
#SBATCH --mem=16G
#SBATCH --time=00:30:00
#SBATCH --array=0-2%3
#SBATCH --output=slurm-%A_%a.out
set -euo pipefail

cd "$SLURM_SUBMIT_DIR"
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate reignflow-env
export OMP_NUM_THREADS="$SLURM_CPUS_PER_TASK"

python -m reignflow --task_name regression --model LSTM --data CAMELS \
  --input_nc_file examples/data_preparation/data/CAMELS_processed/CAMELS.nc \
  --station_ids 01013500,01022500,01030500 \
  --train_date_list 1999-10-01,2000-09-30 \
  --val_date_list 2000-10-01,2001-09-30 \
  --test_date_list 2001-10-01,2002-09-30 \
  --seq_len 30 --pred_len 1 --d_model 32 --batch_size 64 --epochs 2 \
  --learning_rate 0.001 --save_best --device cuda:0 \
  --seed "$SLURM_ARRAY_TASK_ID" --des "seed-$SLURM_ARRAY_TASK_ID"
```

Submit with `sbatch train-array.sbatch`. For 19 tasks, `--array=0-18%19`
allows up to 19 simultaneous array members; actual concurrency depends on
resources and account limits. Different models/configurations need their own
commands or an explicit index-to-command mapping. A seed array alone does not
create different experiment recipes. See [Slurm arrays](https://slurm.schedmd.com/job_array.html).

## Check status

```bash
squeue --me
squeue --me --array
```

After a job leaves the queue, use its actual job ID, for example:

```bash
sacct -j 123456 --format=JobID,JobName,State,ExitCode,Elapsed,MaxRSS
```

`PD` means pending and `R` means running. A completed scheduler job should
also have exit code `0:0` and the expected final ReignFlow artifacts. Preserve
Slurm logs for errors that are not copied to `results.txt`.

The development checkout also has a NERSC-specific launcher and
`nersc/RUNBOOK.md`; they are not included in this documentation preview.
Their site assumptions are separate from the TTU template above. This page's
batch template was syntax/CLI checked in the development checkout; the preview
build does not submit jobs.
