# Local Environments

Two package managers are available globally on the cluster for creating user-managed software environments: **micromamba** (a fast, conda-compatible tool) and **pixi** (a project-based tool built on conda-forge and PyPI).

Use these when you need packages not available as modules, or when you need a pinned, reproducible environment for a specific project.

!!! warning "Mind your home quota"
    Conda environments can be very large (several GB). Store them in `/storage/projects/<project>/envs/`, not in your home directory. Instructions below show how to set the correct path for each tool.

---

## micromamba

[micromamba](https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html) is a fast, minimal drop-in replacement for `conda`. It reads the same `environment.yml` files and uses the same channel ecosystem (conda-forge, bioconda, etc.).

### Shell setup

micromamba needs to be initialised for your shell once. If it is already initialised (i.e. `micromamba` works and you see a `(base)` prompt or can activate envs), skip this step.

```bash
micromamba shell init --shell bash --root-prefix ~/micromamba
source ~/.bashrc
```

### Create an environment

Always specify the environment path explicitly to avoid filling your home directory:

```bash
micromamba create -p /storage/projects/<project>/envs/myenv python=3.11
```

Or use an `environment.yml` file:

```bash
micromamba create -p /storage/projects/<project>/envs/myenv -f environment.yml
```

Example `environment.yml`:

```yaml
name: myenv
channels:
  - conda-forge
  - bioconda
dependencies:
  - python=3.11
  - numpy
  - pandas
  - samtools=1.20
  - pip:
      - some-pypi-package
```

### Activate and use

```bash
micromamba activate /storage/projects/<project>/envs/myenv

python --version
which python

micromamba deactivate
```

### Manage packages

```bash
# Install additional packages into an active environment
micromamba install -c conda-forge scikit-learn

# Install from PyPI via pip
pip install somepackage

# List installed packages
micromamba list

# Update all packages
micromamba update --all
```

### Remove an environment

```bash
micromamba env remove -p /storage/projects/<project>/envs/myenv
```

### Use in a Slurm job

Activate the environment inside your job script with the full path — do not rely on `~/.bashrc` being sourced automatically:

```bash
#!/bin/bash
#SBATCH --job-name=micromamba-job
#SBATCH --partition=cpu
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=04:00:00

# Activate micromamba
eval "$(micromamba shell hook --shell bash)"
micromamba activate /storage/projects/<project>/envs/myenv

python train.py
```

---

## pixi

[pixi](https://pixi.sh) is a project-based package manager. Instead of a shared environment, each project gets its own `pixi.toml` manifest and a local `.pixi/` directory. Dependencies are locked in `pixi.lock` for full reproducibility.

pixi resolves packages from conda-forge and PyPI. It is well-suited for projects that you want to be fully self-contained and shareable.

### Create a new project

Run this inside your project directory under `/storage/projects`:

```bash
cd /storage/projects/<project>
pixi init .
```

This creates:

```text
.
├── pixi.toml      # dependency manifest (commit this)
└── pixi.lock      # exact locked versions (commit this)
```

### Add dependencies

```bash
# From conda-forge (default)
pixi add python=3.11 numpy pandas

# From bioconda
pixi add --channel bioconda samtools

# From PyPI
pixi add --pypi torch torchvision
```

Dependencies are written to `pixi.toml` and the lock file is updated automatically.

### Run commands

```bash
# Run a command inside the pixi environment
pixi run python train.py
pixi run python --version

# Start an interactive shell inside the environment
pixi shell
```

### Add tasks (optional)

You can define project tasks in `pixi.toml` for common operations:

```toml
[tasks]
train = "python scripts/train.py"
test  = "pytest tests/"
```

Then run:

```bash
pixi run train
pixi run test
```

### Install an existing project

When cloning a project that already has `pixi.toml` and `pixi.lock`, restore the exact environment with:

```bash
pixi install
```

This reads the lock file and installs the pinned versions — no version drift.

### Use in a Slurm job

```bash
#!/bin/bash
#SBATCH --job-name=pixi-job
#SBATCH --partition=cpu
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=04:00:00

cd /storage/projects/<project>

pixi run python train.py
```

`pixi run` automatically uses the project's locked environment without any shell initialisation step.

!!! tip "Keep `.pixi/` in your project directory"
    By default, pixi stores the environment in `.pixi/envs/` inside the project directory. If your project is under `/storage/projects`, the environment lives there automatically — no home quota impact.

---

## Choosing between micromamba and pixi

| | micromamba | pixi |
|---|---|---|
| Style | Named/path environments | Per-project, lock-file based |
| Compatible with `environment.yml` | Yes | No (uses `pixi.toml`) |
| Reproducibility | Manual (export + pin) | Automatic (`pixi.lock`) |
| Best for | Reusing existing conda setups | New projects where reproducibility matters |
| Works with bioconda | Yes | Yes (`--channel bioconda`) |
| Works with PyPI | Via `pip` | Native (`--pypi`) |
