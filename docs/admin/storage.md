# Storage Administration

!!! danger "Admin only"
    This section is intended for cluster administrators.

---

## Layout

```text
/storage                   150 TB ZFS (storagepool — HDD RAIDZ2)
├── home/                  per-user ZFS datasets, quota enforced
├── projects/              shared project data (cbcrg group writable)
├── datasets/              reference datasets (read-only for users)
├── software/              modules, environments, containers (admin managed)
└── shared/                templates, docs (admin writable, world readable)

/scratch                   14 TB NVMe (fastpool — mirror)
└── $USER/                 per-user temporary space, no backup, auto-purged
```

---

## ZFS pool health

```bash
zpool status               # overall pool health
zfs list                   # dataset list and usage
df -h /storage /scratch    # filesystem-level usage
```

Monthly scrub (add to root crontab):

```cron
0 3 1 * * /usr/sbin/zpool scrub storagepool
```

Check scrub progress:

```bash
zpool status storagepool
```

---

## Top-level datasets

Create once during initial setup:

```bash
sudo zfs create storagepool/home
sudo zfs create storagepool/projects
sudo zfs create storagepool/datasets
sudo zfs create storagepool/software
sudo zfs create storagepool/shared
```

Recommended `recordsize` settings:

```bash
sudo zfs set recordsize=128K storagepool/home       # scripts and configs
sudo zfs set recordsize=1M   storagepool/projects   # large genomics files
sudo zfs set recordsize=1M   storagepool/datasets
```

---

## Permissions

### `/storage/projects` — group writable

```bash
sudo chgrp cbcrg /storage/projects
sudo chmod 2775 /storage/projects
sudo setfacl -m  g:cbcrg:rwx /storage/projects
sudo setfacl -d -m g:cbcrg:rwx /storage/projects
```

Fix ownership on existing content:

```bash
sudo chgrp -R cbcrg /storage/projects
sudo chmod -R g+rwX /storage/projects
sudo find /storage/projects -type d -exec chmod g+s {} \;
```

### `/storage/shared` — admin write, world read

```bash
sudo chown -R root:hpcadmin /storage/shared
sudo chmod -R 2775 /storage/shared
sudo setfacl -m  g:hpcadmin:rwx /storage/shared
sudo setfacl -d -m g:hpcadmin:rwx /storage/shared
sudo setfacl -m  o::rx  /storage/shared
sudo setfacl -d -m o::rx  /storage/shared
```

### `/scratch` — sticky bit, world writable

```bash
sudo chmod 1777 /scratch
```

---

## Provisioning a user home

Run automatically by `create_new_user.sh`, but can be done manually:

```bash
sudo zfs create storagepool/home/USERNAME
sudo zfs set mountpoint=/storage/home/USERNAME storagepool/home/USERNAME
sudo zfs set quota=30G storagepool/home/USERNAME
sudo chown USERNAME:cbcrg /storage/home/USERNAME
sudo chmod 700 /storage/home/USERNAME
```

---

## Quotas

| Path | Default quota | Notes |
|---|---|---|
| `/storage/home/$USER` | 30 GB (adjustable) | Hard ZFS quota per user |
| `/storage/projects` | None | Shared pool; monitor usage |
| `/scratch` | None | Auto-purged; not backed up |

Get or set a quota:

```bash
zfs get quota storagepool/home/USERNAME
sudo zfs set quota=50G storagepool/home/USERNAME
```

---

## Usage reporting

```bash
zfs list -o name,used,avail,refer                # ZFS-level usage
du -sh /storage/home/* 2>/dev/null | sort -h     # per-user home usage
du -sh /storage/projects/* 2>/dev/null | sort -h # per-project usage
du -sh /scratch/* 2>/dev/null | sort -h          # scratch usage by user
```

---

## Scratch auto-purge

The cleanup script removes files older than N days from `/scratch`:

```bash
sudo /opt/hpc-admin/scripts/clean_scratch.sh 30
```

Daily cron (root crontab):

```cron
0 3 * * * /opt/hpc-admin/scripts/clean_scratch.sh 30 >> /var/log/scratch-cleanup.log 2>&1
```

---

## Multi-node considerations

- Keep `/scratch` local to each compute node (do not share across nodes).
- Share `/storage` via NFS from the primary node, or replicate selected datasets.
- Keep `/storage/software` identical across nodes (NFS mount or sync script).
- Use consistent UID/GID across all nodes — see [User Administration](users.md#audit-and-inventory).
