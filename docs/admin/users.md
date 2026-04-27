# User Administration

!!! danger "Admin only"
    This section is intended for cluster administrators. Do not share admin credentials or run these commands as a regular user.

---

## Groups

| Group | Purpose |
|---|---|
| `cbcrg` | Main lab research users |
| `gpu` | GPU partition access |
| `visitor` | Restricted / student / guest accounts |
| `hpcadmin` | Cluster administrators |

Create a group if it does not exist:

```bash
sudo groupadd -f hpcadmin
```

---

## Create a user

The admin script creates a Linux account, a ZFS home dataset, a scratch directory, and applies the default home quota in one step:

```bash
sudo /opt/hpc-admin/scripts/create_new_user.sh USERNAME [GROUP] [HOME_QUOTA] [SSH_PUBLIC_KEY_FILE]

# Example
sudo /opt/hpc-admin/scripts/create_new_user.sh azizk cbcrg 30G azizk.pub
```

**Verify the home directory:**

```bash
getent passwd USERNAME
su - USERNAME -c pwd
# Expected: /storage/home/USERNAME
```

---

## Manage group membership

```bash
# Add to a group
sudo usermod -aG cbcrg USERNAME
sudo usermod -aG gpu USERNAME      # grant GPU partition access

# Remove from a group
sudo gpasswd -d USERNAME GROUPNAME

# Verify
id USERNAME
```

The user must log out and back in for group changes to take effect.

---

## Lock, unlock, and delete accounts

=== "Lock"

    ```bash
    sudo usermod -L USERNAME       # lock password login
    sudo chage -E 0 USERNAME       # expire the account
    ```

=== "Unlock"

    ```bash
    sudo usermod -U USERNAME
    sudo chage -E -1 USERNAME      # remove expiry
    ```

=== "Delete"

    ```bash
    sudo userdel USERNAME

    # Only destroy the home ZFS dataset after backup or explicit approval
    sudo zfs destroy storagepool/home/USERNAME
    ```

!!! warning "Deleting home data is irreversible"
    Archive or transfer data to `/storage/projects` before destroying the ZFS dataset.

---

## Audit and inventory

```bash
# List all regular user accounts (UID 1000–59999)
getent passwd | awk -F: '$3 >= 1000 && $3 < 60000 {print $1, $3, $4, $6}'

# Export UID/GID map (needed when adding nodes)
getent passwd | awk -F: '$3 >= 1000 && $3 < 60000 {print $1,$3,$4,$6}' \
    > /root/karakoram_users.txt
getent group  | awk -F: '$3 >= 1000 && $3 < 60000 {print $1,$3}' \
    > /root/karakoram_groups.txt
```

!!! tip "UID/GID consistency across nodes"
    Keep identical UID/GID values on every node added to the cluster. For larger deployments consider centralised identity management (FreeIPA or LDAP).
