# Storage Guide

## Overview

| Path | Size | Backed up | Purpose |
|---|---|---|---|
| `/storage` | 150 TB (ZFS) | Yes (ZFS snapshots) | All persistent data |
| `/storage/home/$USER` | quota-limited | Yes | Personal scripts and configs |
| `/storage/projects` | shared pool | Yes | Project data and final outputs |
| `/storage/datasets` | shared pool | Yes | Reference datasets |
| `/storage/software` | admin-managed | Yes | Modules, containers, shared tools |
| `/storage/shared` | admin-managed | Yes | Job templates, documentation |
| `/scratch/$USER` | 14 TB total | **No** | Fast temporary job I/O |

## Home directory

Each user's home is:

```bash
/storage/home/$USER
```

Use home for personal scripts, config files, and small outputs. A hard quota is enforced — check your usage with:

```bash
quota -s
```

!!! warning "Home quota"
    Do not store large datasets, raw sequencing data, or model checkpoints in your home directory. Move bulk data to `/storage/projects`. See [Quotas & Backups](quotas.md) for limits.

## Project storage

```bash
/storage/projects/<username>/<project-name>
```

Use project storage for:

- Long-term experimental results and final outputs
- Data shared with other lab members
- Anything you want to persist after a job completes

There is no per-user quota on project storage, but it is a shared resource — be considerate about disk usage.

## Reference datasets

```bash
/storage/datasets
```

Admin-curated reference genomes, public databases, and shared datasets live here. Do not write to this path. If you need a new dataset added, contact the admin.

## Scratch

```bash
/scratch/$USER/$SLURM_JOB_ID
```

Scratch is a fast NVMe volume intended for temporary files during a job. Use it for:

- Intermediate pipeline outputs
- Large temporary files that would be slow on ZFS

!!! danger "Scratch is not backed up and is auto-purged"
    Files on `/scratch` are **not backed up** and are automatically deleted after a set retention period. Always copy important results to `/storage/projects` before your job ends. See [Quotas & Backups](quotas.md) for the purge schedule.

## Recommended workflow

```bash
# 1. Stage inputs from storage to scratch at job start
cp /storage/projects/myproject/input.fa /scratch/$USER/$SLURM_JOB_ID/

# 2. Run analysis on scratch
cd /scratch/$USER/$SLURM_JOB_ID
my_tool input.fa > output.bam

# 3. Copy results back to storage before job ends
cp output.bam /storage/projects/myproject/results/
```
