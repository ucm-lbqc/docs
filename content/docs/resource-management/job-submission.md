---
title: Job submission
next: job-management
weight: 702
---

Write a script, submit it with `sbatch`, or grab a node interactively with `salloc` / `srun`.
The first complete tutorial is [Getting Started]({{< relref "getting-started" >}}).

## Resource allocation

### sbatch

`sbatch script.slurm` queues a **batch** job.
The script must start with `#!/bin/bash` and Slurm directives (`#SBATCH`).

Useful options:

| Option | Meaning |
| ------ | ------- |
| `-J name` | Job name |
| `-c N` / `--cpus-per-task=N` | CPUs for the task |
| `--mem=32G` | RAM (default is ~4 GB per CPU) |
| `-t 08:00:00` | Wall time (`14-00:00:00` is the partition max) |
| `-o %x-%j.out` | Stdout file (`%j` job id, `%x` name) |
| `-e %x-%j.err` | Stderr file |
| `--gpus=l4:1` | GPU type and count |
| `--constraint=epyc-7713` | Node feature |
| `--mail-user=` / `--mail-type=` | [Mail]({{< relref "slurm#notification-system" >}}) |

#### Job script

Load modules **in the script**.
Use `$SCRATCH_DIR` for heavy I/O.
Exit with the application's status so Slurm records failure correctly.

#### Features

`sinfo -o '%f'` lists features (`cpu-amd`, `gpu-l4`, `xeon-silver-4216`, …).
Combine with `--constraint=` when you need a specific ISA or GPU family.

### salloc

Allocates resources and **holds** them for interactive use:

```console
$ salloc -c 8 --mem=16G -t 02:00:00
```

When the allocation starts, you get a shell **on the login node** with Slurm env vars set.
Start work on the compute node with `srun` (see [Interactive jobs]({{< relref "job-interactive" >}})).

### srun

Runs a step **inside** an allocation (or creates one if you pass resource flags).

```console
$ srun -c 4 -t 00:10:00 hostname
```

Inside a batch script, `srun ./app` is the right way to launch MPI.

## Parallel jobs

> [!WARNING]
> **One node.** MPI ranks must fit on a single machine (up to 128 cores on rose/maria).
> Multi-node `mpirun` will not get a second node.

### Affinity

Hyper-threading is disabled. Bind MPI to cores:

```bash
srun --cpu-bind=cores -n 16 ./mpi_app
```

### OpenMP

```bash
#SBATCH -c 16
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
```

Do not let the runtime see all 128 cores of rose if you only reserved 8.

## GPUs

| Node | GRES |
| ---- | ---- |
| sina | `gpu:l4:2` |
| vision | `gpu:2080ti:2,gpu:3090:1` |
| wc01 | `gpu:2080ti:1,gpu:2080super:1` |

```bash
#SBATCH --gpus=l4:1
#SBATCH --gpus=2080ti:1
#SBATCH --gpus=1
```

The last form takes any free GPU.
CUDA modules: `module load cuda` (and the application module).

## Environment variables

Slurm sets (among others):

| Variable | Content |
| -------- | ------- |
| `$SLURM_JOB_ID` | Job id |
| `$SLURM_JOB_NAME` | Name |
| `$SLURM_CPUS_PER_TASK` | CPUs from `-c` |
| `$SLURM_JOB_NODELIST` | Node |
| `$SLURM_SUBMIT_DIR` | Directory where you ran `sbatch` |
| `$SLURM_GPUS_ON_NODE` | GPUs on the node (if any) |

`$DATA_DIR` and `$SCRATCH_DIR` come from the cluster login environment, not Slurm; they are still set in batch jobs.

## Examples

### Serial / multithreaded CPU

See the genomics scripts under [Software]({{< relref "software" >}}) (FastQC, STAR, TElocal).
Typical header:

```bash
#SBATCH -J star-align
#SBATCH -c 16
#SBATCH --mem=32G
#SBATCH -t 08:00:00
```

### MPI

```bash
#SBATCH -n 32
#SBATCH --mem=64G
#SBATCH -t 12:00:00
module load gnu12 openmpi4
srun --cpu-bind=cores ./mpi_app
```

Keep `-n` ≤ cores on one node.

### GPU

The GROMACS walk-through in [Getting Started]({{< relref "getting-started" >}}) (`--gpus=2080ti:1`, scratch, `USR1` trap) is the reference GPU batch script.
