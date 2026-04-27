# FAQ & Troubleshooting

??? question "My job is stuck in `PD` (pending) — why won't it start?"

    Check the reason code with:

    ```bash
    squeue -u $USER -o "%.18i %.9P %.8j %.8T %.10R"
    ```

    Common reasons:

    | Reason | Meaning | Fix |
    |---|---|---|
    | `Resources` | Requested resources are not yet free | Wait — resources will be allocated when available |
    | `Priority` | Higher-priority jobs are ahead | Wait |
    | `QOSMaxCpuPerUserLimit` | You have hit a per-user CPU cap | Finish or cancel existing jobs first |
    | `ReqNodeNotAvail` | Requested a node that is down | Remove `--nodelist` constraint if set |
    | `InvalidQOS` | Wrong partition name | Check `sinfo` for valid partition names |

    If a job has been pending for more than a few hours with no explanation, contact the admin.

??? question "My job was killed — what happened?"

    Check the job's exit code and reason:

    ```bash
    scontrol show job JOBID | grep -E "JobState|ExitCode|Reason"
    ```

    | Exit code / reason | Cause | Fix |
    |---|---|---|
    | `OUT_OF_MEMORY` | Job exceeded `--mem` limit | Increase `--mem` and resubmit |
    | `TIMEOUT` | Job hit the wall-clock limit | Increase `--time` or checkpoint and resume |
    | `CANCELLED` | Job was cancelled by you or the admin | Check email for admin notice |
    | Non-zero exit code | Your script crashed | Check the `.err` / `.out` log files |

??? question "`module: command not found` or module not loading"

    Lmod is sourced automatically at login. If it is missing, your shell may not have sourced `/etc/profile`:

    ```bash
    source /etc/profile
    ```

    If a specific module is not found:

    ```bash
    module spider nextflow      # search for it — names are case-sensitive
    module avail                # browse all available modules
    ```

    If the module you need does not exist, contact the admin to request it.

??? question "SSH connection refused or timeout"

    - Confirm you are on the MBZUAI network (or VPN if working remotely).
    - Check that your SSH key is in `~/.ssh/authorized_keys` on the cluster. If you recently regenerated your key, send the new public key to the admin.
    - Try with verbose output to see where it hangs: `ssh -v USERNAME@karakoram.mbzu.ae`
    - If the login node is unresponsive, contact the admin — it may be in maintenance.

??? question "My scratch files are gone"

    `/scratch` is **not backed up** and files are automatically purged after a retention period. If your files disappeared:

    - They were either purged by the auto-cleanup job, or
    - The scratch volume was reset after maintenance.

    Always copy important results to `/storage/projects` inside your job script **before the job ends**. See [Storage Guide](../storage/storage.md) for the recommended workflow.

??? question "`nvidia-smi: command not found` inside my job"

    You are likely running on the CPU partition. GPU tools are only available on the GPU partition:

    ```bash
    #SBATCH --partition=gpu
    #SBATCH --gres=gpu:1
    ```

    Also confirm the CUDA module is loaded if your code needs it:

    ```bash
    module load cuda/system
    nvidia-smi
    ```

??? question "My home directory is full — writes are failing"

    Check what is using space:

    ```bash
    du -sh /storage/home/$USER/*  | sort -h
    ```

    Common culprits: micromamba environments, pip caches, large log files, model checkpoints.

    Quick clean-up targets:

    ```bash
    # Remove pip cache
    rm -rf ~/.cache/pip

    # Remove cached micromamba packages (keeps environments intact)
    micromamba clean --all

    # Move large files to project storage
    mv /storage/home/$USER/bigdata /storage/projects/myproject/
    ```

    If you need a larger quota, contact the admin.

??? question "How do I run a Jupyter notebook on the cluster?"

    See [Interactive Jobs → Jupyter notebook over SSH](../slurm/interactive.md#jupyter-notebook-over-ssh) for the full step-by-step guide.

??? question "Can I use micromamba on the cluster?"

    Yes. micromamba is installed globally and is the recommended way to manage conda-style environments. conda and mamba are **not** available — use micromamba instead.

    Always create environments under `/storage/projects` to avoid filling your home quota:

    ```bash
    micromamba create -p /storage/projects/<project>/envs/myenv python=3.11
    micromamba activate /storage/projects/<project>/envs/myenv
    ```

    In Slurm job scripts, activate with the shell hook rather than relying on `~/.bashrc`:

    ```bash
    eval "$(micromamba shell hook --shell bash)"
    micromamba activate /storage/projects/<project>/envs/myenv
    ```

    For full usage see [Local Environments](../software/environments.md#micromamba).
