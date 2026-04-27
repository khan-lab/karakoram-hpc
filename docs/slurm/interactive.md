# Interactive Jobs

Interactive sessions let you run commands directly on a compute node instead of submitting a batch script. Use them for testing, debugging, or exploratory work — not for long-running production jobs.

!!! warning "Do not work interactively on the login node"
    All interactive work that uses significant CPU, memory, or GPU must be allocated through Slurm using `srun`.

## Interactive shell

=== "CPU"

    ```bash
    srun --partition=cpu --cpus-per-task=4 --mem=16G --time=02:00:00 --pty bash
    ```

=== "GPU"

    ```bash
    srun --partition=gpu --gres=gpu:1 --cpus-per-task=4 --mem=16G --time=02:00:00 --pty bash
    ```

Once allocated, you will get a shell prompt on the compute node. Type `exit` to release the allocation when done.

## Jupyter notebook over SSH

Running Jupyter requires two steps: start the server on a compute node, then tunnel the port to your laptop.

**Step 1 — start an interactive session and launch Jupyter:**

```bash
# On the login node: request a compute allocation
srun --partition=cpu --cpus-per-task=4 --mem=16G --time=04:00:00 --pty bash

# On the compute node: load Python and start Jupyter (no browser)
module load python/3.11   # use the appropriate module name
jupyter notebook --no-browser --port=8888
```

Note the URL and token printed in the terminal (e.g. `http://127.0.0.1:8888/?token=abc123`).

**Step 2 — open an SSH tunnel from your local machine:**

```bash
# Replace NODE with the compute node name (shown in the srun prompt, e.g. compute01)
# Replace USERNAME with your cluster username
ssh -N -L 8888:NODE:8888 USERNAME@karakoram.mbzu.ae
```

Open `http://127.0.0.1:8888/?token=abc123` in your local browser.

!!! tip
    Find the node name by running `hostname` inside the `srun` session, or check `squeue` from another terminal.

## VS Code Remote

You can use VS Code to edit files and submit jobs without leaving your IDE.

1. Install the [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) extension in VS Code.
2. Add `karakoram` as a host (see [Account Setup](../getting-started/account-setup.md#configure-your-ssh-client)).
3. Connect: **Remote Explorer → karakoram → Connect**.
4. Open a terminal inside VS Code and use `sbatch` / `srun` as normal.

!!! warning
    VS Code connects to the **login node**. Do not run compute-intensive processes in the VS Code terminal. Submit jobs with `sbatch` or start an `srun` session for interactive work.
