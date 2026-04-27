# Cluster Policy

## Compute

- All heavy jobs must be submitted through Slurm — no exceptions.
- CPU jobs must use the `cpu` partition (`--partition=cpu`).
- GPU jobs must use the `gpu` partition and request the GPU explicitly (`--gres=gpu:1`).
- The cluster has **1 GPU**. Request only `--gres=gpu:1`; do not request more.
- Interactive work that uses significant CPU or memory must use `srun`, not the login shell.
- Do not run long-running processes on the login node. Such processes will be terminated without notice.

## Storage

- Use `/storage/home/$USER` for personal scripts, configs, and small files only. A hard quota is enforced.
- Use `/storage/projects` for project data, shared outputs, and anything that needs to persist long-term.
- Use `/scratch/$USER` for temporary job files during a run. Copy results to `/storage/projects` before the job ends.
- Do not treat `/scratch` as permanent storage. It is auto-purged and not backed up.
- Do not write to `/storage/datasets` or `/storage/software` — these are admin-managed.

## Software

- Load software via modules (`module load`) wherever possible.
- Use Apptainer/Singularity for containers not available as modules.
- Docker is reserved for admin-managed services. Do not install or run Docker as a user.
- Do not install software system-wide. Install to your home directory or ask the admin to add a module.

## Resource etiquette

- Request only the resources your job actually needs. Over-requesting blocks other users.
- Use `seff JOBID` after jobs complete and tune your requests accordingly.
- If you need to run many jobs simultaneously, use job arrays (`--array`) rather than hundreds of individual submissions.
- If you need dedicated or extended resources, contact the admin in advance.

## Data and reproducibility

- Copy important results from scratch to `/storage/projects` at the end of every job.
- Use modules or container images (not ad-hoc pip/conda installs) to ensure your workflows are reproducible.
- Document your workflows — Nextflow, Snakemake, or shell scripts with pinned module versions are all acceptable.

## Violations

Repeated violations — running heavy jobs on the login node, filling shared storage without notice, or hoarding the GPU — may result in your jobs being cancelled and your account suspended pending a conversation with the admin.
