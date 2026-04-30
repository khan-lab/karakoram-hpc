# Quotas & Backups

## Home directory quota

A hard disk quota is enforced on `/storage/home/$USER`. When you hit the hard limit, writes will fail.

Check your current usage:

```bash
quota -s
```

The output shows your usage, soft limit (warning threshold), and hard limit. Contact the admin if you need the exact limit values for your account.

**What to do when your home is full:**

1. Find large files: `du -sh /storage/home/$USER/* | sort -h`
2. Delete files you no longer need
3. Move datasets and results to `/storage/projects`
4. Compress archives: `tar -czf archive.tar.gz mydirectory/`

## Project storage

`/storage/projects` has no per-user quota. It is a shared resource managed collectively by the lab. Coordinate with your PI or collaborators before writing large amounts of data.

## Scratch purge policy

`/scratch` is a **14 TB** NVMe volume shared across all users. It is **not backed up**.

!!! danger "Auto-purge"
    Files on `/scratch` older than **[30 days — confirm with admin]** are automatically deleted without warning. Do not use scratch as long-term storage. Copy results to `/storage/projects` at the end of every job.

To see how much scratch you are using:

```bash
du -sh /scratch/$USER
```

## Backup policy

| Path | Backup method | Retention |
|---|---|---|
| `/storage/home` | ZFS snapshots | Contact admin for schedule |
| `/storage/projects` | ZFS snapshots | Contact admin for schedule |
| `/storage/datasets` | ZFS snapshots | Contact admin for schedule |
| `/scratch` | **None** | Not backed up |

ZFS snapshots are point-in-time copies. If you accidentally delete a file in `/storage`, the admin may be able to restore it from a snapshot — contact them as soon as possible.

## Checking disk usage

```bash
du -sh /storage/home/$USER              # your home usage
du -sh /storage/projects/$USER/myproject      # a specific project
du -sh /scratch/$USER                   # your scratch usage
df -h /storage                          # total storage fill level
df -h /scratch                          # total scratch fill level
```
