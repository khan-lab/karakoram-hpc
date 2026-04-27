# Slurm Administration

!!! danger "Admin only"
    This section is intended for cluster administrators.

---

## Key files

| File | Purpose |
|---|---|
| `/etc/slurm/slurm.conf` | Main Slurm configuration |
| `/etc/slurm/gres.conf` | GPU/generic resource definitions |
| `/etc/munge/munge.key` | Shared authentication key (identical on all nodes) |
| `/var/log/slurm/slurmctld.log` | Controller daemon log |
| `/var/log/slurm/slurmd.log` | Compute daemon log |

---

## Status checks

```bash
sinfo                           # partition and node state
squeue                          # all queued and running jobs
scontrol show node              # full node details
scontrol show config            # active Slurm configuration
```

---

## Services

=== "MUNGE"

    ```bash
    systemctl status munge
    munge -n | unmunge          # verify MUNGE is working end-to-end
    sudo systemctl restart munge
    ```

=== "Slurm daemons"

    ```bash
    sudo systemctl restart slurmctld   # controller (head node)
    sudo systemctl restart slurmd      # compute daemon
    sudo systemctl status slurmctld slurmd
    ```

---

## Partition configuration

Example stanzas in `/etc/slurm/slurm.conf`:

```conf
PartitionName=cpu   Nodes=NODE Default=YES  MaxTime=7-00:00:00  State=UP
PartitionName=gpu   Nodes=NODE AllowGroups=gpu MaxTime=7-00:00:00 State=UP
PartitionName=debug Nodes=NODE              MaxTime=01:00:00    State=UP
PartitionName=long  Nodes=NODE              MaxTime=14-00:00:00 State=UP
```

After editing `slurm.conf`, reload without full restart:

```bash
sudo scontrol reconfigure
```

---

## GPU resource (GRES)

`/etc/slurm/gres.conf`:

```conf
NodeName=NODE Name=gpu File=/dev/nvidia0
```

Verify the GPU is visible to Slurm:

```bash
nvidia-smi
srun -p gpu --gres=gpu:1 nvidia-smi
```

---

## Accounting (simple setup)

For a single-lab cluster without multi-account accounting, disable enforcement to avoid `InvalidAccount` errors:

```conf
AccountingStorageType=accounting_storage/none
AccountingStorageEnforce=none
PriorityType=priority/basic
```

If a job is stuck with reason `InvalidAccount`, check the job script for an `#SBATCH --account` line and remove it, or align it with `scontrol show config | grep -i account`.

---

## Node maintenance

```bash
# Drain a node (let running jobs finish, prevent new ones)
sudo scontrol update NodeName=$(hostname -s) State=DRAIN Reason="maintenance"

# Resume after maintenance
sudo scontrol update NodeName=$(hostname -s) State=RESUME

# Force a node back to idle (use with caution)
sudo scontrol update NodeName=$(hostname -s) State=IDLE
```

---

## Test jobs

```bash
# Quick CPU test
srun -p cpu hostname

# Quick GPU test
srun -p gpu --gres=gpu:1 nvidia-smi

# Full template jobs
sbatch /storage/shared/templates/cpu_job.sh
sbatch /storage/shared/templates/gpu_job.sh
```

---

## Logs

```bash
sudo tail -f /var/log/slurm/slurmctld.log   # follow controller log
sudo tail -f /var/log/slurm/slurmd.log      # follow compute log
sudo grep -i error /var/log/slurm/slurmctld.log | tail -50
```

---

## Hostname or node changes

If the node hostname changes, update all three files:

```text
/etc/slurm/slurm.conf
/etc/slurm/gres.conf
/etc/hosts
```

Then restart in order:

```bash
sudo systemctl restart munge
sudo systemctl restart slurmctld slurmd
```

---

## Multi-node expansion

For each new compute node:

1. Copy the MUNGE key (must be identical on all nodes):

    ```bash
    sudo scp /etc/munge/munge.key NODE:/etc/munge/
    sudo ssh NODE "chown munge:munge /etc/munge/munge.key && chmod 400 /etc/munge/munge.key"
    sudo ssh NODE "systemctl restart munge"
    ```

2. Distribute `slurm.conf` and `gres.conf` to the new node.
3. Ensure the same UID/GID mapping — see [User Administration](users.md#audit-and-inventory).
4. Enable and start `slurmd` on the new node:

    ```bash
    sudo ssh NODE "systemctl enable --now slurmd"
    ```

5. Add the node and update partitions in `slurm.conf`, then `sudo scontrol reconfigure`.
