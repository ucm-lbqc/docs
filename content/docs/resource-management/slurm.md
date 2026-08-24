---
title: SLURM
prev: resource-management
next: job-submission
weight: 701
---

## What is Slurm?

Slurm is the batch scheduler.
You describe CPUs, memory, GPUs, and time; it places the job on a free compute node and runs your script there.

The login node (`lbqc`) is **not** a Slurm compute node.
`sbatch` from `lbqc` is correct; running the analysis binaries on `lbqc` is not.

## Partitions

There is a single partition:

| Partition | Default | Max nodes | Max time | Nodes |
| --------- | ------- | --------- | -------- | ----- |
| `normal` | yes | **1** | **14 days** | all compute nodes |

You do not need `#SBATCH -p` unless you want to be explicit (`-p normal`).

> [!WARNING]
> `MaxNodes=1`: a job cannot span two machines.
> Ask for many cores on one node (`#SBATCH -c 32`), not `-N 2`.

## QoS

No extra Quality-of-Service names are configured (`QoS=N/A`).
Fair-share and node `Weight` still apply: cheaper/smaller nodes are preferred when they fit.

## Notification system

Email when a job starts, ends, or fails:

```bash
#SBATCH --mail-user=you@ucm.cl
#SBATCH --mail-type=ALL
```

Use an address you read. `BEGIN`, `END`, `FAIL`, `TIME_LIMIT` are valid subsets of `--mail-type`.

## Scheduling

Slurm uses **consumable cores and memory** (`CR_Core_Memory`) and prefers the **least-loaded** eligible node (`CR_LLN`).
Each node has a `Weight` so light Intel boxes (nc01–04) are chosen before large EPYC nodes when both match.

### Priority

Jobs wait if nothing matches, or if higher-priority work is queued.
You cannot jump the queue; shrink time, CPUs, or memory if you are stuck in `(Resources)` or `(Priority)`.

### Aging

Waiting jobs gain priority over time (standard Slurm aging).
There is nothing to configure as a user.

### Features and constraints

```console
$ sinfo -N -o '%N %f %G'
```

Examples:

```bash
#SBATCH --constraint=cpu-amd
#SBATCH --gpus=l4:1
```

See [Cluster]({{< relref "cluster" >}}) for CPU/GPU inventory.
