---
title: Interactive Jobs
next: best-practices
weight: 704
---

Use an interactive allocation when you need a shell on a compute node (tests, short GUI-less debugging, building software).
It still **counts as a job**: CPUs, memory, GPUs, and time are reserved.

## srun --pty (simplest)

From the login node:

```console
$ srun --pty -c 8 --mem=16G -t 02:00:00 bash
```

When a node is free, you land in a bash on that node.
`module load`, run commands, `exit` to release the allocation.

GPU example:

```console
$ srun --pty --gpus=l4:1 -c 4 --mem=16G -t 01:00:00 bash
```

## salloc + srun

```console
$ salloc -c 8 --mem=16G -t 02:00:00
$ srun --pty bash
```

`salloc` may leave you on the login node with Slurm variables set until you `srun`.
Always start the heavy process through `srun` so it runs on the compute node.

## Notes

- Interactive jobs wait in the queue like batch jobs. Off-peak is easier.
- Do not hold a 128-core allocation idle.
- X11 apps are not supported in a useful way; use batch or copy files out.
- Same **one-node** limit as `sbatch`.

Walkthrough context: [Getting Started]({{< relref "getting-started" >}}).
