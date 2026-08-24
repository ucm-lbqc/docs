---
title: LBQC cluster
next: accounts
weight: 200
---

The LBQC runs a high-performance computing (HPC) cluster: one **login node** (`lbqc`) and ten **compute nodes** scheduled by Slurm.

Funding:

- Faculty of Medicine, Universidad Católica del Maule, Chile.
- ANID (SIA SA772100091), Chile.

## Login and compute nodes

Use the login node only to edit files, transfer data, and submit jobs.
All CPU- or GPU-heavy work must run on a compute node via Slurm (`sbatch`, `salloc`, or `srun`).
See [Getting Started]({{< relref "getting-started" >}}) and [Job submission]({{< relref "job-submission" >}}).

The default partition is **`normal`**. Every job is limited to **one node**. The maximum wall time is **14 days**.

## Hardware

| Name | CPU | Cores | RAM | GPU |
| ---- | --- | ----- | --- | --- |
| login (`lbqc`) | Intel Xeon E-2276G @ 3.80 GHz | 6 | 64 GB | |
| nc01–nc04 | 2× Intel Xeon Silver 4216 @ 2.1 GHz | 32 | 256 GB | |
| nc05 | AMD EPYC 9534 @ 2.45 GHz | 64 | 512 GB | |
| maria | 2× AMD EPYC 7713 @ 2.0 GHz | 128 | 320 GB | |
| rose | 2× AMD EPYC 7713 @ 2.0 GHz | 128 | 512 GB | |
| sina | 2× Intel Xeon Silver 4416+ @ 2.0 GHz | 40 | 512 GB | 2× NVIDIA L4 |
| vision | Intel Xeon Silver 4116 | 24 | 56 GB | 2× GeForce RTX 2080 Ti, 1× RTX 3090 |
| wc01 | Intel Xeon Silver 4216 @ 2.10 GHz | 16 | 128 GB | GeForce RTX 2080 Ti, RTX 2080 Super |

Compute nodes: **10 nodes**, **528 CPU cores**, about **3 TB** RAM, **8 GPUs**.
Power is backed by 2× APC Smart-UPS On-Line 10 kVA.

Request a GPU with `#SBATCH --gpus=<type>:<count>` (for example `--gpus=l4:1` or `--gpus=2080ti:1`).
CPU features can be selected with `#SBATCH --constraint=` (see `sinfo -o '%f'`).

## Storage

User storage is described in [Storage]({{< relref "storage" >}}). The login node also exports the module tree:

- **HOME** `/home/<user>` — NFS RAID 1 on the login node (2× 6 TB). Not a backup.
- **Software** `/opt/ohpc/pub` — NFS RAID 1 (2× 2 TB), modules for everyone.
- **Data** `$DATA_DIR` — QNAP TS-673A NAS, RAID 6, NFS. Long-term working data.
- **Scratch** `$SCRATCH_DIR` — local disk on the compute node. Deleted after a week.

## Software stack (system)

- Base OS: Rocky Linux 9.5
- Provisioning: Warewulf 4
- Scheduler: Slurm
- Software environment: OpenHPC 3 + Lmod
