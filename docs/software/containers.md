# Containers

The cluster runs [Apptainer](https://apptainer.org/) (the open-source successor to Singularity) for containerised workflows. Apptainer lets you run any Docker or OCI image without root privileges, making it safe for shared HPC systems.

!!! note "Docker is admin-only"
    Docker requires root access and is reserved for admin-managed services. Use Apptainer for all user-facing container workflows.

---

## Lab biocontainers

The CBCRG lab maintains a set of pre-built, versioned container images on GitHub:

**[https://github.com/khan-lab/biocontainers](https://github.com/khan-lab/biocontainers)**

Images are published to the GitHub Container Registry (`ghcr.io/khan-lab/`). Always prefer these over pulling arbitrary images from Docker Hub — they are tested, pinned, and built for the lab's workflows.

Browse available images in the repository and use them as:

```bash
apptainer exec --bind /storage,/scratch \
    docker://ghcr.io/khan-lab/<image>:<tag> \
    <command>
```

Replace `<image>` and `<tag>` with the name and version from the [biocontainers releases](https://github.com/khan-lab/biocontainers/releases).

---

## Pulling an image

Pull an image once and cache it locally to avoid re-downloading on every job:

```bash
# Pull from the lab registry
apptainer pull /storage/software/containers/myimage.sif \
    docker://ghcr.io/khan-lab/<image>:<tag>

# Pull from Docker Hub
apptainer pull /storage/software/containers/ubuntu24.sif \
    docker://ubuntu:24.04
```

!!! tip "Use `/storage/software/containers/` for shared images"
    Store images that multiple lab members will use under `/storage/software/containers/` (admin can help set permissions). Store personal or experimental images under `/storage/projects/$USER/<project>/containers/`.

---

## Running containers

### Execute a single command

```bash
apptainer exec --bind /storage,/scratch \
    /path/to/image.sif \
    python script.py
```

### Interactive shell inside a container

```bash
apptainer shell --bind /storage,/scratch \
    /path/to/image.sif
```

### Run the container's default entrypoint

```bash
apptainer run --bind /storage,/scratch \
    /path/to/image.sif
```

---

## Bind mounts

By default, only your home directory is visible inside the container. Use `--bind` to expose additional paths:

```bash
# Expose storage and scratch
apptainer exec --bind /storage,/scratch image.sif bash

# Bind with a different internal path
apptainer exec --bind /storage/projects/$USER/myproject:/data image.sif bash
# /data inside the container now points to /storage/projects/$USER/myproject
```

Always bind at least `/storage` for project data access. Add `/scratch` when the job reads or writes temporary files there.

---

## GPU access inside containers

Apptainer can expose the host GPU to the container using `--nv` (NVIDIA):

```bash
apptainer exec --nv --bind /storage,/scratch \
    docker://ghcr.io/khan-lab/<gpu-image>:<tag> \
    python train.py
```

`--nv` automatically maps the CUDA libraries from the host into the container — you do not need to install CUDA inside the image.

---

## Using containers in Slurm jobs

### CPU job

```bash
#!/bin/bash
#SBATCH --job-name=container-job
#SBATCH --partition=cpu
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=04:00:00
#SBATCH --output=%j.out

IMAGE=/storage/software/containers/myimage.sif

apptainer exec --bind /storage,/scratch \
    "$IMAGE" \
    python /storage/projects/$USER/myproject/scripts/run.py
```

### GPU job

```bash
#!/bin/bash
#SBATCH --job-name=gpu-container-job
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=08:00:00
#SBATCH --output=%j.out

IMAGE=/storage/software/containers/myimage.sif

apptainer exec --nv --bind /storage,/scratch \
    "$IMAGE" \
    python /storage/projects/$USER/myproject/scripts/train.py
```

---

## Environment variables inside containers

Pass environment variables with `--env` or `--env-file`:

```bash
apptainer exec \
    --env MY_VAR=value \
    --env-file /storage/projects/$USER/myproject/.env \
    image.sif python script.py
```

---

## Building images (local only)

If you need a custom image, build it on your **local machine** (not the cluster — Apptainer build requires root or fakeroot):

```bash
# On your local machine — write a definition file
cat > myimage.def << 'EOF'
Bootstrap: docker
From: ghcr.io/khan-lab/base:<tag>

%post
    pip install mypackage==1.2.3
EOF

sudo apptainer build myimage.sif myimage.def
```

Transfer the `.sif` to the cluster:

```bash
scp myimage.sif karakoram:/storage/projects/$USER/myproject/containers/
```

For images that will be shared across the lab, open a pull request on [khan-lab/biocontainers](https://github.com/khan-lab/biocontainers) instead.
