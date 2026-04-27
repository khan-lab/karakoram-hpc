# Karakoram HPC <small>Documentation</small>
> **Karakoram** is a High-Performance Computing (HPC) cluster for large-scale data analysis at the Computational Biology and Cancer Regulatory Genomics (CBCRG) group at MBZUAI, Abu Dhabi, UAE.


!!! info
    This server is exclusively for the CBCRG lab members.


## Hardware

| Resource | Specification |
|---|---|
| CPU cores | 384 total |
| GPU | 1× NVIDIA RTX 6000 Ada Generation |
| Storage (`/storage`) | 150 TB ZFS |
| Scratch (`/scratch`) | 14 TB NVMe — **not backed up** |
| Job scheduler | Slurm |

## Features

- **Slurm** job scheduler with `cpu` and `gpu` partitions
- **ZFS storage** with snapshots — home, projects, datasets, and software
- **Fast scratch** on `/scratch` for temporary job I/O
- **Lmod** software modules for reproducible environments
- **Apptainer/Singularity** for containerised workflows
- **SSH access** from anywhere on the MBZUAI network

!!! warning "Login node rule"
    The login node (`karakoram.mbzu.ae`) is for submitting jobs, editing scripts, and transferring files only. Do not run analyses, training runs, or anything CPU/memory-intensive on it. Use Slurm.

## Quick commands

```bash
sinfo                                        # show partitions and node status
squeue                                       # show running/pending jobs
module avail                                 # list available software
sbatch /storage/shared/templates/cpu_job.sh # submit an example CPU job
```

## Storage layout

```text
/storage/home/$USER      personal home (quota enforced)
/storage/projects        shared project data and final outputs
/storage/datasets        reference datasets and public data
/storage/software        admin-managed software, modules, containers
/storage/shared          job templates and shared documentation
/scratch/$USER           fast temporary scratch (auto-purged)
```
