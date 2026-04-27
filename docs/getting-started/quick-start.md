# Quick Start

!!! tip "New user?"
    If you do not have an account yet, start with the [Account Setup](account-setup.md) guide first.

## 1. Log in

```bash
ssh USERNAME@karakoram.mbzu.ae
```

Replace `USERNAME` with your cluster username. You will need your SSH key configured — see [Account Setup](account-setup.md).

## 2. Check cluster status

```bash
sinfo                          # partition and node overview
sinfo -o "%P %C %G"            # show CPUs and GPUs per partition
squeue                         # all running and pending jobs
squeue -u $USER                # your jobs only
```

## 3. Check your storage

```bash
ls /storage/home/$USER         # your home directory
ls /storage/projects           # shared project space
quota -s                       # check your home quota usage
```

## 4. Load software

```bash
module avail                   # list all available modules
module spider nextflow         # search for a specific module
module load nextflow/latest    # load a module (replace with actual name)
module list                    # confirm what is loaded
```

## 5. Submit a job

=== "CPU job"

    ```bash
    sbatch /storage/shared/templates/cpu_job.sh
    ```

=== "GPU job"

    ```bash
    sbatch /storage/shared/templates/gpu_job.sh
    ```

## 6. Monitor and manage jobs

```bash
squeue -u $USER                # your jobs
scontrol show job JOBID        # full details for a job
scancel JOBID                  # cancel a job
seff JOBID                     # efficiency report after job completes
```

!!! warning "No heavy work on the login node"
    Use Slurm for everything compute-intensive. Jobs running directly on the login node will be terminated.
