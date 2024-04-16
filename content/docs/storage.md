---
title: Storage
next: software
weight: 500
---

## Distributed storage

As with any Unix-based OS, you will be placed at your home directory (`/home/$USER`) upon successful login, where `$USER` is your account's username.
Here you will store your personal files and configurations.
You can reference your home directory by the `$HOME` environment variable or the `~` shorthand in a path.

{{< callout type="info" >}}
HOME directories are stored in a redundant array of independent disks (RAID) configured with mirroring to create a large reliable data store shared across users.
However, **HOME directories are not backed up** and have limited capacity.
Be sure to keep your home directory clean by regularly deleting old data or by moving data to another location.
{{< /callout >}}

You will also have 1 TB at your disposal located at the `/data/$USER` directory, which is backed up regularly.
These data directories are stored in a high-performance 6-bay NAS server and are aimed at storing important and long-term data.

HOME and data directories are shared over the network, invisible to the end user, such that all files and directories are always available on all cluster nodes.
Therefore, you do not need to copy your files between nodes each time you run a job.[^1]

For actual work, however, you should use the local **scratch directory**.
This space is stored in a physical disk on the compute nodes so it is much faster for reading and writing files than the shared file system over the network.
Using the scratch directory avoids slowing down calculations, especially if they read and write large intermediate files (>1GB).

{{< callout type="warning" >}}
Old files (more than one week without access) in the scratch directory are automatically deleted so always be sure to move the output files back to either your HOME or data directory.
{{< /callout >}}

Refer to the [Best practices]() page for more information about how to correctly use the file system.

### NFS node (home & opt)

### NAS

### Scratch

## File organization

## Quota

## File transfer

### scp

### rsync
