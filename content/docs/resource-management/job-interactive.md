---
title: Interactive Jobs
next: best-practices
weight: 704
---

An **interactive** job still goes through Slurm: you wait in the queue, then you get a shell (or a single command) **on a compute node**.
Use it to test a command, compile software, or debug, not to hold a large allocation idle.

Batch scripts (`sbatch`) are described in [Job submission]({{< relref "job-submission" >}}).
This page is only the case where **you** type commands live.

Two ways:

1. `salloc` — reserve a node, stay on the login node, then `srun` onto the compute node.
2. `srun --pty` — one command that both reserves a node and opens a shell there.

Both count against the same limits as batch jobs: **one node**, CPUs, memory, GPUs, and a time limit.

## `salloc`: reserve, then work interactively

`salloc` asks Slurm for resources and **holds** them until you exit.
On this cluster the new prompt is still on the **login node**.
The allocation is real — `squeue` shows the job as `R` — but `hostname` is still `lbqc` until you start a step with `srun`.

```console
[user@lbqc ~]$ salloc -c 4 --mem-per-cpu=4G -t 01:00:00
salloc: Pending job allocation 312
salloc: job 312 queued and waiting for resources
salloc: job 312 has been allocated resources
salloc: Granted job allocation 312
salloc: Waiting for resource configuration
salloc: Nodes nc01 are ready for job
[user@lbqc ~]$ hostname
lbqc
[user@lbqc ~]$ echo "job $SLURM_JOB_ID on $SLURM_JOB_NODELIST"
job 312 on nc01
[user@lbqc ~]$ squeue --me
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
               312    normal interact  user     R       0:08      1 nc01
```

Heavy commands must run through `srun`, otherwise they still execute on `lbqc`:

```console
[user@lbqc ~]$ srun hostname
nc01
[user@lbqc ~]$ srun --pty bash
[user@nc01 ~]$ module load gnu12
[user@nc01 ~]$ which gcc
/opt/ohpc/pub/.../gcc
[user@nc01 ~]$ exit
exit
[user@lbqc ~]$ exit
salloc: Relinquishing job allocation 312
```

`srun --pty bash` gives you an interactive Bash **on the compute node** (prompt host changes).
Plain `srun hostname` runs one command on the compute node and returns.

GPU example:

```console
[user@lbqc ~]$ salloc --gpus=l4:1 -c 4 --mem-per-cpu=4G -t 01:00:00
[user@lbqc ~]$ srun --pty bash
[user@sina ~]$ module load cuda
```

> [!WARNING]
> Until you `exit` the `salloc` session (or `scancel` the job id), the node stays reserved.
> Do not leave an allocation sitting idle.

## `srun --pty`: shell in one step

If you only want a prompt on a compute node, `srun --pty` both creates the allocation and starts Bash there.
You skip the “still on lbqc” stage.

```console
[user@lbqc ~]$ srun --pty -c 4 --mem-per-cpu=4G -t 01:00:00 bash
[user@nc01 ~]$ hostname
nc01
[user@nc01 ~]$ module load gnu12
[user@nc01 ~]$ exit
```

When the shell exits, the allocation is released.

GPU:

```console
[user@lbqc ~]$ srun --pty --gpus=2080ti:1 -c 4 --mem-per-cpu=4G -t 01:00:00 bash
```

This is the same `srun` as in a batch script, but started **from the login node with resource flags**, so Slurm creates a new job for you.
Inside a file you already submitted with `sbatch`, do **not** repeat `-c` / `-t` on `srun`; the job already has them ([Job submission]({{< relref "job-submission#srun" >}})).

## When to use which

| Goal | Command |
| ---- | ------- |
| Unattended production run | `sbatch script.slurm` |
| Several commands on the same reserved node, typing as you go | `salloc` … then `srun --pty bash` |
| One interactive shell as quickly as possible | `srun --pty … bash` |
| One-off command on a compute node without a script | `srun -c 4 --mem-per-cpu=4G -t 00:10:00 hostname` |

The last line is useful to check that scheduling works; it is not how you run a four-hour analysis (write a batch script).

## Notes

- Interactive jobs wait in the queue like batch jobs. Off-peak is easier.
- X11 graphical apps are not supported in a useful way; copy files out and plot locally.
- Same **one-node** limit as `sbatch`.
- Load modules **after** you are on the compute node (inside `srun --pty`), not only in the SSH session on `lbqc`.
