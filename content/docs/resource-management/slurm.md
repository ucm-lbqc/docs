---
title: SLURM
prev: resource-management
next: job-submission
weight: 701
---

[Getting Started]({{< relref "getting-started" >}}) already showed a first `sbatch`.
This page explains **what the scheduler is doing** so the options in [Job submission]({{< relref "job-submission" >}}) make sense.

## What is Slurm?

Slurm is the program that owns the compute nodes.
You never pick a free machine by hand: you describe how many CPUs, how much memory, whether you need a GPU, and how long the job may run.
Slurm then:

1. Finds a node that can satisfy that request (or waits until one is free).
2. Starts your script on that node.
3. Kills the job if it exceeds the time or memory you asked for, so it cannot starve other users.

The login node (`lbqc`) is **not** a compute node.
`sbatch` **from** `lbqc` is correct; running the analysis **on** `lbqc` is not.

## Partitions

A **partition** is a named pool of nodes with common limits (maximum time, maximum number of machines per job, and so on).
This cluster has a single partition:

| Partition | Default | Max nodes | Max time | Nodes |
| --------- | ------- | --------- | -------- | ----- |
| `normal` | yes | **1** | **14 days** | all compute nodes |

You do not need `#SBATCH -p normal`; that is already the default.

> [!WARNING]
> A job cannot span two machines (`MaxNodes=1`).
> Ask for many cores on **one** node (`#SBATCH -c 32`), not `-N 2`.
> If you need more work done, submit several jobs.

Time is written as `days-hours:minutes:seconds`. These are equivalent:

```bash
#SBATCH -t 08:00:00          # 8 hours
#SBATCH -t 1-00:00:00        # 1 day
#SBATCH -t 14-00:00:00       # partition maximum
```

If the job is still running when the limit is reached, Slurm sends a signal and then kills it.
Shorter limits are easier to schedule: an 8-hour job can fit in a gap that a 14-day job cannot.

## QoS

No extra Quality-of-Service names are configured (`QoS=N/A`).
You do not pass `--qos`.
Fair-share and per-node `Weight` still apply: smaller Intel nodes (`nc01`–`nc04`) are preferred when they have enough resources, so a large EPYC machine is not taken by a 4-core job if a small node is free.

## Notification system

Slurm can email you when a job starts, ends, or fails:

```bash
#SBATCH --mail-user=you@ucm.cl
#SBATCH --mail-type=ALL
```

Use an address you actually read.
`--mail-type` can be a subset instead of `ALL`: `BEGIN`, `END`, `FAIL`, `TIME_LIMIT`.
Mail is a convenience, not a guarantee that the job succeeded; always check the output file and `sacct`.

## How a node is chosen

Slurm treats **cores and memory as consumable**: if `rose` has 128 cores and your job takes 32, 96 remain for other jobs (subject to memory).
It prefers the **least-loaded** eligible node.

You normally do **not** name a node.
Naming one (`#SBATCH -w rose`) makes the job wait until *that* machine is free, even if another identical one is idle.

### Priority and aging

Jobs wait if nothing matches, or if higher-priority work is queued.
Waiting jobs slowly gain priority (aging).
You cannot jump the queue; you can only make the job easier to place by asking for less time, fewer CPUs, or less memory.

Pending reasons are listed with example `squeue` output in [Job management]({{< relref "job-management#pending" >}}).

### Features (tags on each node) {#features}

Every compute node has a list of **features**: short tags that describe the CPU family, the vendor, optional GPUs, and similar facts.
They exist so you can say “I need an AMD CPU” or “I need a node that has an L4 GPU” without memorizing hostnames.

You can ignore features for most jobs.
Slurm will pick any node that has enough CPUs, memory, and GPUs.
Use `--constraint` only when the program really needs a given kind of machine (a binary compiled for a particular CPU, or a GPU model).

How to list features and how to write `--constraint` is in [Job submission]({{< relref "job-submission#features" >}}).
The hardware table is in [Cluster]({{< relref "cluster" >}}).
