---
title: Job Management
next: job-interactive
weight: 703
---

## squeue

List your jobs:

```console
$ squeue --me
$ squeue -j 238
```

Common states: `PD` pending, `R` running, `CG` completing.
Pending reasons: [Troubleshooting]({{< relref "troubleshooting" >}}).

## sinfo

```console
$ sinfo
$ sinfo -N -l
$ sinfo -N -o '%N %c %m %G %T %E'
```

`idle`, `mix`, `alloc`, `drain` describe node state.
`%E` is the reason if a node is down.

## scontrol

```console
$ scontrol show job 238
$ scontrol show node rose
$ scontrol show partition
```

Use this when `squeue` is not enough (requested vs granted memory, features, stderr path).

## sacct

Accounting for **finished** jobs (and running ones):

```console
$ sacct -j 238 --format=JobID,JobName,State,Elapsed,MaxRSS,ExitCode
$ sacct --me --starttime=2026-08-01
```

`MaxRSS` is useful after an OOM kill (`OUT_OF_MEMORY` / `TIMEOUT`).

## scancel

```console
$ scancel 238
$ scancel --me
$ scancel -n star-align
```

`scancel --me` cancels **all** your jobs; there is no undo.

## Logging into a compute node

Slurm PAM blocks SSH unless you already have a **running** job on that node.

Prefer:

```console
$ srun --pty -c 4 --mem=8G -t 01:00:00 bash
```

or `salloc` then `srun --pty bash` ([Interactive jobs]({{< relref "job-interactive" >}})).

If a batch job is running, `ssh <node>` from the login node may work for debugging **that** job only.
Do not use it to start extra compute.
