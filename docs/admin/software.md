# Software Administration

!!! danger "Admin only"
    This section is intended for cluster administrators.

---

## Software model

| Layer | Tool | Scope |
|---|---|---|
| Modules | Lmod | System-wide, user-facing |
| Conda environments | micromamba | Shared or per-user |
| Project environments | pixi | Per-project, user-managed |
| Containers | Apptainer | User-facing research containers |
| Services | Docker | Admin-managed only |

---

## Directory layout

```text
/storage/software/
├── apps/         installed application binaries
├── bin/          admin helper scripts
├── containers/   shared Apptainer .sif images
├── envs/         micromamba environments backing modules
├── modules/      Lmod modulefiles
└── pixi/         shared pixi home (PIXI_HOME)
```

Initial setup:

```bash
sudo mkdir -p /storage/software/{apps,bin,containers,envs,modules,pixi}
sudo chgrp -R hpcadmin /storage/software
sudo chmod -R 2775 /storage/software
```

---

## Lmod

Install:

```bash
sudo apt update && sudo apt install -y lmod
```

Enable for all users via profile.d:

```bash
echo 'source /usr/share/lmod/lmod/init/bash' \
    | sudo tee /etc/profile.d/00-lmod.sh
echo 'module use /storage/software/modules' \
    | sudo tee /etc/profile.d/01-hpc-modules.sh
```

Verify:

```bash
source /etc/profile
module avail
```

### Module tree

```text
/storage/software/modules/
├── bio/1.0.lua
├── cuda/system.lua
├── nextflow/latest.lua
├── nf-core/latest.lua
├── python/3.11.lua
└── R/4.5.0.lua
```

### Add a module

Create a directory and write a Lua modulefile:

```bash
sudo mkdir -p /storage/software/modules/tool
sudo nano /storage/software/modules/tool/1.0.lua
```

Minimal modulefile:

```lua
help([[
Brief description of the tool.
]])

whatis("Name: tool")
whatis("Version: 1.0")

prepend_path("PATH", "/storage/software/envs/tool/bin")
```

Test:

```bash
module spider tool
module load tool/1.0
which tool
```

---

## micromamba — shared environments

Create a shared environment that backs a module:

```bash
micromamba create -y \
    -p /storage/software/envs/myenv \
    -c conda-forge -c bioconda \
    PACKAGE1 PACKAGE2
```

Install additional packages into an existing environment:

```bash
micromamba install -y \
    -p /storage/software/envs/myenv \
    -c conda-forge PACKAGE
```

!!! note
    Do not symlink `conda` or `mamba` to `micromamba`. Users should call `micromamba` directly.

---

## Module recipes

### Python

```text
environment : /storage/software/envs/python-3.11
modulefile  : /storage/software/modules/python/3.11.lua
```

Test:

```bash
module load python/3.11
python --version
jupyter --version
```

### R / Bioconductor

```text
environment : /storage/software/envs/R-4.5.0
modulefile  : /storage/software/modules/R/4.5.0.lua
```

Test:

```bash
module load R/4.5.0
Rscript -e 'library(GenomicRanges); sessionInfo()'
```

### Bioinformatics CLI tools (`bio/1.0`)

```text
environment : /storage/software/envs/bio
modulefile  : /storage/software/modules/bio/1.0.lua
```

Typical contents: `samtools`, `bcftools`, `bedtools`, `bwa`, `minimap2`, `fastqc`, `multiqc`, `gatk4`, `strelka`, `manta`, `delly`, `cnvkit`, `mosdepth`, `picard`, `snpEff`, `VEP`.

Test:

```bash
module load bio/1.0
samtools --version
bcftools --version
gatk --version
```

### Nextflow / nf-core

```bash
module load nextflow/latest
module load nf-core/latest
nextflow -version
nf-core --version
```

Shared Nextflow config (used with `-profile slurm,apptainer`):

```text
/storage/software/nextflow-config/nextflow.config
```

### CUDA

Only expose a CUDA module if the full toolkit (not just the driver) is installed:

```bash
nvidia-smi                        # confirm driver is present
ls /usr/local/ | grep cuda        # confirm toolkit is installed
nvcc --version                    # confirm nvcc is available
```

If only the driver is present, the `cuda/system` module should expose CUDA libraries from `/usr/local/cuda` without exposing `nvcc`.

---

## pixi — global configuration

Set a shared pixi home so the cache and global tools are not scattered across user homes:

```bash
echo 'export PIXI_HOME=/storage/software/pixi' \
    | sudo tee /etc/profile.d/02-pixi.sh
```

Users should set their own pixi cache to scratch to avoid quota issues:

```bash
# Recommended addition to user ~/.bashrc (document in onboarding)
export PIXI_CACHE_DIR=/scratch/$USER/pixi-cache
```

---

## Apptainer

Set a shared cache location for image pulls:

```bash
echo 'export APPTAINER_CACHEDIR=/scratch/$USER/apptainer-cache' \
    | sudo tee -a /etc/profile.d/02-pixi.sh
```

Store shared images in:

```text
/storage/software/containers/
```

Pull a lab image from the GitHub Container Registry:

```bash
apptainer pull /storage/software/containers/myimage.sif \
    docker://ghcr.io/khan-lab/<image>:<tag>
```

---

## Docker (admin services only)

```bash
getent group docker                  # list users in the docker group
sudo docker ps                       # show running containers
sudo docker compose up -d            # start admin services
```

!!! warning
    Regular users must not be in the `docker` group. Use Apptainer for all user-facing container workflows.

---

## Multi-node software sync

If `/storage/software` is not NFS-mounted on compute nodes, sync it manually:

```bash
sudo /storage/software/bin/sync_software_to_node.sh NODE_HOSTNAME
```

For production multi-node setups, mount `/storage/software` via NFS so all nodes see the same modules and environments without manual sync.
