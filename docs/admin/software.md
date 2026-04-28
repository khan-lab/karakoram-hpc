# Software Administration

!!! danger "Admin only"
    This section is intended for cluster administrators.

---

## Software model

| Layer | Tool | Scope |
|---|---|---|
| Modules | Lmod | System-wide, user-facing |
| Conda environments | micromamba | Shared environments backing modules |
| Project environments | pixi | Per-project, user-managed |
| Containers | Apptainer | User-facing research containers |
| Services | Docker | Admin-managed only |

---

## Directory layout

```text
/storage/software/
├── apps/         installed binary application skeletons
├── bin/          admin helper scripts
├── containers/   shared Apptainer .sif images
├── envs/         micromamba environments backing modules
├── manifests/    installation log (software.tsv)
├── modules/      Lmod modulefiles
└── pixi/         shared pixi home (PIXI_HOME)
```

Initial setup:

```bash
sudo mkdir -p /storage/software/{apps,bin,containers,envs,manifests,modules,pixi}
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

---

## Installing modules

Use the installer script for all new software. It creates the micromamba environment, writes the Lmod modulefile, creates a `latest` symlink, and records the installation in the manifest — all in one step.

```text
/opt/hpc-admin/scripts/install_module_tool.sh
```

### Usage

```bash
sudo /opt/hpc-admin/scripts/install_module_tool.sh TOOL VERSION TYPE [PACKAGE]
```

| Argument | Description |
|---|---|
| `TOOL` | Module name users will `module load` |
| `VERSION` | Version string (used for the modulefile and environment path) |
| `TYPE` | Installation method — see table below |
| `PACKAGE` | Package identifier passed to the installer (defaults to `TOOL` if omitted) |

### Install types

| Type | What it does | When to use |
|---|---|---|
| `pypi` | Creates a Python 3.11 env and installs the package from PyPI via pip | Python tools distributed on PyPI |
| `conda` | Creates a micromamba env with a single conda/bioconda package at an exact version | Any conda-forge or bioconda tool |
| `bio` | Creates a micromamba env with a space-separated list of conda/bioconda packages | Bioinformatics tool bundles |
| `binary` | Creates an empty `apps/TOOL/VERSION/bin/` skeleton and a modulefile | Pre-compiled binaries installed manually |

### Examples

=== "PyPI"

    ```bash
    sudo /opt/hpc-admin/scripts/install_module_tool.sh encodefetch 0.3.0 pypi encodefetch
    ```

=== "Conda (single)"

    ```bash
    sudo /opt/hpc-admin/scripts/install_module_tool.sh samtools 1.21 conda samtools
    ```

=== "Bio bundle"

    ```bash
    sudo /opt/hpc-admin/scripts/install_module_tool.sh variant-tools 1.0 bio \
        'samtools bcftools bedtools gatk4 cnvkit delly manta survivor'
    ```

=== "Binary"

    ```bash
    sudo /opt/hpc-admin/scripts/install_module_tool.sh mytool 2.3.1 binary
    # Place binaries in: /storage/software/apps/mytool/2.3.1/bin/
    ```

After a successful install the script prints:

```text
module load TOOL/VERSION
module load TOOL/latest
```

Both work — `latest` is a symlink to the most recently installed version.

### Reinstalling

If the tool/version already exists the script will prompt for confirmation before removing the existing environment and modulefile. To skip the prompt:

```bash
FORCE=1 sudo /opt/hpc-admin/scripts/install_module_tool.sh samtools 1.21 conda samtools
```

### Verify after install

```bash
module spider TOOL          # confirm Lmod sees it
module load TOOL/VERSION
which TOOL_BINARY
TOOL_BINARY --version
module unload TOOL/VERSION
```

---

## Installation manifest

Every install is appended to:

```text
/storage/software/manifests/software.tsv
```

Columns: `tool`, `version`, `type`, `package`, `install_path`, `module_file`, `installed_at`.

Query the manifest:

```bash
column -t /storage/software/manifests/software.tsv   # pretty-print
grep samtools /storage/software/manifests/software.tsv
```

---

## Module recipes

### Python

```bash
sudo /opt/hpc-admin/scripts/install_module_tool.sh python 3.11 pypi python
```

Test:

```bash
module load python/3.11
python --version
jupyter --version
```

### R / Bioconductor

```bash
sudo /opt/hpc-admin/scripts/install_module_tool.sh R 4.5.3 conda r-base
```

!!! note
    R and Bioconductor environments often need additional packages after the initial install. Add them with:
    ```bash
    micromamba install -y -p /storage/software/envs/R/4.5.3 -c conda-forge -c bioconda r-biocmanager bioconductor-genomicranges
    ```

Test:

```bash
module load R/4.5.3
Rscript -e 'library(GenomicRanges); sessionInfo()'
```

### Bioinformatics CLI tools

```bash
sudo /opt/hpc-admin/scripts/install_module_tool.sh bio 1.0 bio \
    'samtools bcftools bedtools bwa minimap2 fastqc multiqc gatk4 \
     strelka manta delly cnvkit mosdepth picard snpeff ensembl-vep'
```

Test:

```bash
module load bio/1.0
samtools --version
bcftools --version
gatk --version
```

### Nextflow

```bash
sudo /opt/hpc-admin/scripts/install_module_tool.sh nextflow 25.10.4 conda nextflow
```

Shared Nextflow config (used with `-profile slurm,apptainer`):

```text
/storage/software/nextflow-config/nextflow.config
```

Test:

```bash
module load nextflow/latest
nextflow -version
```

### CUDA

Only create a CUDA module if the full toolkit (not just the driver) is installed:

```bash
nvidia-smi                        # confirm driver is present
ls /usr/local/ | grep cuda        # confirm toolkit is installed
nvcc --version                    # confirm nvcc is available
```

The `binary` type works well here — point the modulefile at the existing system CUDA path rather than installing into an env:

```bash
sudo /opt/hpc-admin/scripts/install_module_tool.sh cuda system binary
# Then manually edit /storage/software/modules/cuda/system.lua
# to prepend /usr/local/cuda/bin and set LD_LIBRARY_PATH
```

---

## Manual modulefiles

For edge cases where the installer does not fit, write a Lmod modulefile directly:

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

prepend_path("PATH",            "/storage/software/envs/tool/1.0/bin")
prepend_path("LD_LIBRARY_PATH", "/storage/software/envs/tool/1.0/lib")
setenv("TOOL_HOME",             "/storage/software/envs/tool/1.0")
```

Create the `latest` symlink manually if needed:

```bash
cd /storage/software/modules/tool
sudo ln -sfn 1.0.lua latest.lua
```

---

## pixi — global configuration

Set a shared pixi home so the global cache is not scattered across user homes:

```bash
echo 'export PIXI_HOME=/storage/software/pixi' \
    | sudo tee /etc/profile.d/02-pixi.sh
```

Users should point their pixi cache to scratch to avoid home quota issues:

```bash
# Add to user ~/.bashrc during onboarding
export PIXI_CACHE_DIR=/scratch/$USER/pixi-cache
```

---

## Apptainer

Set a per-user cache location via profile.d:

```bash
echo 'export APPTAINER_CACHEDIR=/scratch/$USER/apptainer-cache' \
    | sudo tee -a /etc/profile.d/02-pixi.sh
```

Store shared images in `/storage/software/containers/`. Pull lab images from the GitHub Container Registry:

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
