# Account Setup

## Request an account

Contact the Aziz to request access. Provide:

- Your full name and MBZUAI email address
- Your intended use (project name or research area)

The admin will create your account and ask for your SSH public key.

## Generate an SSH key

If you do not already have an SSH key, generate one:

```bash
ssh-keygen -t ed25519 -C "your.email@mbzuai.ac.ae"
```

Accept the default path (`~/.ssh/id_ed25519`) and set a passphrase. Your **public key** is the file ending in `.pub`:

```bash
cat ~/.ssh/id_ed25519.pub
```

Send this public key string to the admin. Never share your private key (`id_ed25519`).

## Configure your SSH client

Add an entry to `~/.ssh/config` on your local machine for convenient access:

```text
Host karakoram
    HostName karakoram.mbzu.ae
    User YOUR_USERNAME
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
```

After saving, you can log in with just:

```bash
ssh karakoram
```

## First login checklist

Once the admin confirms your account is ready:

```bash
# 1. Log in
ssh karakoram

# 2. Verify your home directory exists
ls /storage/home/$USER

# 3. Check your storage quota
quota -s

# 4. Check cluster partitions
sinfo

# 5. Confirm you can load a module
module avail
```

If any of these steps fail, contact the admin with the error message.

## Transferring files

Use `scp` or `rsync` to move data to and from the cluster:

```bash
# Copy a file to the cluster
scp localfile.txt karakoram:/storage/home/$USER/

# Copy a directory recursively
rsync -avz myproject/ karakoram:/storage/projects/myproject/

# Copy results back to your local machine
scp karakoram:/storage/projects/myproject/results.tar.gz .
```

!!! tip
    Use `/storage/projects` for data shared with collaborators and `/storage/home/$USER` for personal scripts and configs only.
