---
title: LBQC cluster
next: accounts
weight: 200
---

The LBQC runs a high-performance computing (HPC) cluster at Universidad Católica del Maule.
It consists of one **login node** that you connect to, and ten **compute nodes** that run the actual work.

Funding:

- Faculty of Medicine, Universidad Católica del Maule, Chile.
- ANID (SIA SA772100091), Chile.

## Login node and compute nodes

The **login node** is named `lbqc`. That is the machine you reach with SSH.
Everyone shares it: it is the place to edit files, copy data in and out, inspect the queue, and submit jobs.
It is a small workstation (6 cores, 64 GB of RAM), not a compute server.
If you run a heavy analysis here, you slow down every other user, and the account can be disabled.
How to submit work to the rest of the cluster is covered in [Getting Started]({{< relref "getting-started" >}}) and [Resource management]({{< relref "resource-management" >}}).

The **compute nodes** are the machines with the large CPUs, memory, and GPUs.
You do not log in to one of them to “just run” a production job.
A scheduler (Slurm) picks a free node that matches the resources you asked for and runs your job there.

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

In total the compute side has **10 nodes**, **528 CPU cores**, about **3 TB** of RAM, and **8 GPUs**.
Power is backed by 2× APC Smart-UPS On-Line 10 kVA.

A **core** is one CPU worker. Asking for 8 cores means your program may use 8 threads (or 8 MPI ranks) on a single machine.
**RAM** is the memory that program can use; if it needs more than you reserved, the job is killed.
**GPUs** are extra accelerators on `sina`, `vision`, and `wc01` only. Requesting them is explained in [Job submission]({{< relref "job-submission#gpus" >}}).

Hyper-threading is disabled: the “Cores” column is the number you can actually request on that node.
The largest machines (`rose`, `maria`) have 128 cores; a job cannot use more than one node.

## Storage (physical)

User-visible directories are documented in [Storage]({{< relref "storage" >}}). Physically:

- **HOME** — mirrored HDDs on the login node (2× 6 TB, RAID 1), exported over NFS.
- **Software** (`/opt/ohpc/pub`) — mirrored HDDs (2× 2 TB, RAID 1), cluster-wide modules. You cannot write here.
- **Data** (`$DATA_DIR`) — QNAP TS-673A NAS, RAID 6, NFS. Long-term working data.
- **Scratch** (`$SCRATCH_DIR`) — local disk on the compute node that ran the job.

RAID is redundancy against a disk failure, not a backup.

## Software stack (system)

- Base OS: Rocky Linux 9.5
- Provisioning: Warewulf 4
- Scheduler: Slurm
- Software environment: OpenHPC 3 + Lmod
