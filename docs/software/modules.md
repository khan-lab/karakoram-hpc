# Software Modules

Karakoram HPC uses [Lmod](https://lmod.readthedocs.io/) to manage software environments. Modules let you switch between software versions without conflicts.

## Browse available software

```bash
module avail                   # list all modules
module avail bio               # filter by category or name prefix
module spider nextflow         # search for a module by keyword
module spider nextflow/25.10.4   # show details for a specific version
```

## Load and unload modules

```bash
module load nextflow/latest    # load a module
module load R/4.5.3        # load another
module list                    # show what is currently loaded
module unload nextflow/latest  # unload a specific module
module purge                   # unload everything
```

## Available module categories

| Category | Example modules | Description |
|---|---|---|
| `bio/` | `bio/1.0` | Bioinformatics tools (samtools, bwa, etc.) |
| `r-bioc/` | `R/4.5.3` | R + Bioconductor packages |
| `nextflow/` | `nextflow/25.10.4` | Nextflow workflow engine |
| `cuda/` | `cuda/system` | CUDA toolkit (GPU partition only) |
| `python/` | `python/3.11` | Python interpreter |

!!! tip
    Module names above are examples. Run `module avail` on the cluster to see the exact names and versions currently installed. Request new software from the admin.

## Save and restore module environments

```bash
module save myenv              # save current set of modules as "myenv"
module restore myenv           # restore a saved environment
module savelist                # list saved environments
```

Use this to reproduce the exact software stack for a project.

!!! tip "Need a container?"
    For containerised workflows see [Containers](containers.md).
