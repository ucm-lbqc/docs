---
title: Storage
next: software
weight: 500
---

The cluster exposes three user-visible file systems.
They differ in speed, lifetime, and how safe they are as a place to leave results.

| Location | Path / variable | Speed | Intended capacity | Lifetime | Backup |
| -------- | --------------- | ----- | ----------------- | -------- | ------ |
| HOME | `$HOME` (`/home/$USER`) | Slow (NFS) | Keep it small (~100 GB guideline) | Account lifetime | No |
| Data | `$DATA_DIR` | Slow (NFS NAS) | ~1 TB guideline | Until the account is removed | No (RAID only) |
| Scratch | `$SCRATCH_DIR` | Fast (local disk) | Shared per node (~1 TB class) | **One week** | No |

`$HOME`, `$DATA_DIR`, and `$SCRATCH_DIR` are set for you at login.

> [!IMPORTANT]
> RAID on HOME or the NAS is **not** a backup.
> The lab is not responsible for data in `/data`.
> Copy anything you cannot afford to lose off the cluster (laptop, extra disk, or another institutional store).

Collaborators (users whose Unix group is a lab member, not themselves) have:

```text
$DATA_DIR=/data/<group>/collab/$USER
```

and a home directory under that member's tree, not `/home/$USER`.

## HOME (NFS)

HOME lives on mirrored HDDs on the login node and is exported over NFS to every compute node.
Use it for dotfiles, small scripts, and job files.
Do not dump FASTQ, BAM, or STAR indices here.

### Software (`/opt/ohpc/pub`)

The same NFS server exports cluster-wide modules.
You cannot write there; request a module instead ([Software]({{< relref "software#request" >}})).

## Data (NAS)

`$DATA_DIR` is on the QNAP NAS (RAID 6) and is mounted on all nodes.
Put **inputs you will reuse** and **results you must keep** here: reference genomes, STAR indices, SRA cache, final BAM/counts, MultiQC reports.

It is still network storage. Do not run heavy random I/O against it during a job; stage large intermediates on scratch.

## Scratch

`$SCRATCH_DIR` is a **local** disk on the compute node assigned to your job.
It is the right place for FASTQ conversion, trimming, alignment temp files, and anything larger than about 1 GB that is only needed for that run.

> [!WARNING]
> Files on scratch that are unused for a week are deleted.
> Copy keepers back to `$DATA_DIR` (or off the cluster) before the job directory ages out.
> Scratch is **not** shared across nodes as a coherent workspace: always use the path on the node that ran the job, typically via `$SCRATCH_DIR` inside the job script.

## File organization

A layout that stays out of the way of NFS and scratch cleanup:

```text
$DATA_DIR/
  ref/                 # genomes, GTF, TElocal .locInd
  reads/               # original FASTQ (or SRA cache under ncbi/)
  results/<sample>/    # final BAM, counts, reports
$HOME/
  jobs/                # *.slurm scripts
$SCRATCH_DIR/
  <jobid>/             # per-job work directory
```

Create the scratch workdir in the job, `cd` into it, then copy products to `$DATA_DIR` at the end (see [Best practices]({{< relref "best-practices" >}})).

## Quota

Hard per-user quotas are **not** enabled yet.
Follow the capacity guidelines above and do not fill HOME or the NAS.
If a node or NFS share is full, jobs fail with `No space left on device`; delete or move data and contact [fadasme@ucm.cl](mailto:fadasme@ucm.cl) if the share looks stuck.

## File transfer {#file-transfer}

Copy from your laptop, not from the login node as a compute step.
`scp` is fine for a few files; `rsync` is better for trees and restarts.

**Copy to the cluster** (run this **locally**):

```console
$ scp /path/to/local-file <user>@{{% data "server.ip" %}}:"$DATA_DIR"/reads/
$ rsync -avh --progress /path/to/local-dir/ <user>@{{% data "server.ip" %}}:"$DATA_DIR"/reads/
```

Remote paths after `:` are relative to HOME unless they start with `/`.
`$DATA_DIR` is an **absolute** path on the cluster; quote it and use the full path, or use `lbqc:` after you set an SSH alias ([Access]({{< relref "access" >}})).

Example with an alias and an absolute data directory:

```console
$ rsync -avh --progress ./fastq/ lbqc:/data/<user>/reads/
```

> [!TIP]
> `rsync -avh --partial` resumes interrupted transfers.
> Prefer compressing on the wire only if the CPU on either side is cheaper than the network (`-z`).
> FASTQ.gz is already compressed; skip `-z` for those.
