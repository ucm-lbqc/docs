---
title: Storage
next: software
weight: 500
---

The cluster gives you three places to keep files.
They look similar in a terminal (`ls` works in all of them), but they are **not** interchangeable: they differ in speed, how much you should store, how long files last, and whether anyone will save them if a disk dies.

| Location | Path / variable | Speed | Intended capacity | Lifetime | Backup |
| -------- | --------------- | ----- | ----------------- | -------- | ------ |
| HOME | `$HOME` | Slow (network disk) | Keep it small (~100 GB guideline) | Account lifetime | No |
| Data | `$DATA_DIR` | Slow (network NAS) | ~1 TB guideline | Until the account is removed | No (RAID only) |
| Scratch | `$SCRATCH_DIR` | Fast (disk on the compute node) | Shared per node (~1 TB class) | **One week** | No |

`$HOME`, `$DATA_DIR`, and `$SCRATCH_DIR` are set for you at login and inside batch jobs.
Use the variables in scripts instead of typing a path that might be wrong for a collaborator account.

> [!IMPORTANT]
> RAID on HOME or the NAS is **not** a backup.
> RAID only survives a disk failure in the same box.
> The lab is not responsible for data in `/data`.
> Copy anything you cannot afford to lose off the cluster (laptop, extra disk, or another institutional store).

## Members vs collaborators

**Members** (Unix group equal to the username):

```text
$HOME=/home/$USER
$DATA_DIR=/data/$USER
```

**Collaborators** of a member named `<member>`:

```text
$HOME=/home/<member>/collab/$USER
$DATA_DIR=/data/<member>/collab/$USER
```

Scratch is `/scratch/$USER` on every compute node in both cases.

## HOME

HOME lives on mirrored hard disks on the login node and is exported over the network (NFS) to every compute node.
Use it for configuration files, small scripts, and job files (`*.slurm`).
Do not store large datasets here: sequencing reads, simulation trajectories, genome indices, and similar files belong on `$DATA_DIR`.

### Software (`/opt/ohpc/pub`)

The same NFS server exports the cluster-wide modules.
You cannot write there; request a module instead ([Software]({{< relref "software#request" >}})).

## Data (NAS)

`$DATA_DIR` is on the QNAP NAS (RAID 6) and is mounted on all nodes.
Put **inputs you will reuse** and **results you must keep** here: reference files, downloaded datasets, final outputs, reports.

It is still network storage.
A job that reads and writes millions of small files, or a huge temporary file, will be slow and will load the NAS for everyone.
Copy large working files to scratch at the start of the job, work there, and copy the keepers back at the end.

## Scratch

`$SCRATCH_DIR` is a **local** disk on the compute node assigned to your job.
It is much faster than HOME or the NAS because the job does not go over the network for every read and write.
Use it for temporary files: unpacked inputs, intermediate results, program working directories.

> [!WARNING]
> Files on scratch that are unused for a week are deleted.
> Copy keepers back to `$DATA_DIR` (or off the cluster) before that.
> Scratch is **not** a shared workspace across nodes: always use `$SCRATCH_DIR` *inside* the job, which points to the disk of the node that is actually running.

## Suggested layout

Keep job scripts in HOME (small) and data in `$DATA_DIR`.
A layout that stays readable as projects grow:

```text
$DATA_DIR/
  inputs/              # original data you will reuse
  results/<run>/       # outputs you must keep
$HOME/
  jobs/                # *.slurm scripts
$SCRATCH_DIR/
  $SLURM_JOB_ID/       # per-job work directory (created in the script)
```

In the job: create the scratch work directory, `cd` into it, run the program, then copy products to `$DATA_DIR` before the script exits.
[Job submission]({{< relref "job-submission" >}}) shows a complete example; [Best practices]({{< relref "best-practices" >}}) explains why.

## Quota

Hard per-user quotas are **not** enabled yet.
Follow the capacity guidelines above and do not fill HOME or the NAS.
If a node or NFS share is full, jobs fail with `No space left on device`; delete or move data and contact [fadasme@ucm.cl](mailto:fadasme@ucm.cl) if the share looks stuck.

## File transfer {#file-transfer}

Copy from your laptop, not from the login node as a compute step.
`scp` is fine for a few files; `rsync` is better for directory trees and for restarting an interrupted copy.

**Copy to the cluster** (run this **locally**, on your computer):

```console
$ scp /path/to/local-file <user>@{{% data "server.ip" %}}:"$DATA_DIR"/inputs/
$ rsync -avh --progress /path/to/local-dir/ <user>@{{% data "server.ip" %}}:"$DATA_DIR"/inputs/
```

Remote paths after `:` are relative to HOME unless they start with `/`.
`$DATA_DIR` is an **absolute** path on the cluster; quote it, or use an SSH alias ([Access]({{< relref "access" >}})) and the full path:

```console
$ rsync -avh --progress ./dataset/ lbqc:/data/<user>/inputs/
```

Collaborators replace `/data/<user>/` with `/data/<member>/collab/<user>/` (or just use `"$DATA_DIR"` after logging in).

> [!TIP]
> `rsync -avh --partial` resumes interrupted transfers.
> Skip `-z` (compression) for files that are already compressed (`.gz`, `.bz2`, `.zip`).
