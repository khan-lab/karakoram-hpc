# Running Jobs with Slurm

All compute work must go through Slurm. Do not run analyses on the login node.

## Common `#SBATCH` options

| Option | Example | Description |
|---|---|---|
| `--job-name` | `--job-name=myrun` | Label shown in `squeue` |
| `--partition` | `--partition=cpu` | `cpu` or `gpu` |
| `--cpus-per-task` | `--cpus-per-task=8` | CPU cores to allocate |
| `--mem` | `--mem=32G` | RAM per node |
| `--time` | `--time=12:00:00` | Wall-clock limit (HH:MM:SS) |
| `--output` | `--output=%j.out` | stdout log file (`%j` = job ID) |
| `--error` | `--error=%j.err` | stderr log file |
| `--mail-type` | `--mail-type=END,FAIL` | Email on completion or failure |
| `--mail-user` | `--mail-user=you@mbzuai.ac.ae` | Email address for notifications |
| `--gres` | `--gres=gpu:1` | GPU resource (GPU partition only) |

## CPU job

```bash
#!/bin/bash
#SBATCH --job-name=cpu-test
#SBATCH --partition=cpu
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=04:00:00
#SBATCH --output=%j.out
#SBATCH --error=%j.err

# Use job-specific scratch directory for temporary files
WORKDIR=/scratch/$USER/$SLURM_JOB_ID
mkdir -p "$WORKDIR"
cd "$WORKDIR"

# Your commands here
hostname
echo "Running with $SLURM_CPUS_PER_TASK CPUs"

# Copy results back before the job ends
cp -r "$WORKDIR/results" /storage/projects/myproject/
```

Submit:

```bash
sbatch script.sh
```

## GPU job

!!! info "GPU availability"
    The cluster has **1 GPU**. Only one job can use it at a time. The maximum request is `--gres=gpu:1`.

```bash
#!/bin/bash
#SBATCH --job-name=gpu-test
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=04:00:00
#SBATCH --output=%j.out
#SBATCH --error=%j.err

nvidia-smi
python train.py
```

## Job arrays

Run the same script for many inputs in parallel:

```bash
#!/bin/bash
#SBATCH --job-name=array-test
#SBATCH --partition=cpu
#SBATCH --array=1-20
#SBATCH --cpus-per-task=2
#SBATCH --mem=8G
#SBATCH --time=02:00:00
#SBATCH --output=%A_%a.out

echo "Task ID: $SLURM_ARRAY_TASK_ID"
python process.py --sample $SLURM_ARRAY_TASK_ID
```

`$SLURM_ARRAY_TASK_ID` holds the index (1–20 above). Use `--array=1-20%5` to cap concurrency at 5 at a time.

## Monitor and manage jobs

```bash
squeue                          # all jobs in the queue
squeue -u $USER                 # your jobs only
scontrol show job JOBID         # full details for a job
scancel JOBID                   # cancel a running or pending job
scancel -u $USER                # cancel all your jobs
```

## Check job efficiency after completion

```bash
seff JOBID
```

`seff` shows CPU and memory efficiency. If memory efficiency is below 50%, reduce `--mem` in future submissions to free resources for others.

## Job output files

By default Slurm writes stdout to `slurm-JOBID.out` in the directory where you ran `sbatch`. Use `--output` and `--error` to customise the paths.

!!! warning "Scratch is not persistent"
    Always copy important results from `/scratch/$USER/$SLURM_JOB_ID` to `/storage/projects` **before the job ends**. Scratch files may be purged automatically.
