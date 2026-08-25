---
title: Job submission
next: job-management
weight: 702
---

This page is the reference for **batch** jobs: a script on disk, submitted with `sbatch`, that runs unattended on a compute node.

- First job you ever submit: [Getting Started]({{< relref "getting-started" >}}).
- Shell on a compute node for tests: [Interactive jobs]({{< relref "job-interactive" >}}) (`salloc` and `srun --pty`).
- What partitions, time limits, and mail mean: [SLURM]({{< relref "slurm" >}}).

## Anatomy of a job script

A job script is a normal Bash file.
The first line tells Linux to run it with Bash.
Lines that start with `#SBATCH` are **not** comments for Bash: Slurm reads them before the script starts and uses them to reserve resources.
Everything after that is executed **on the compute node**, in order.

```bash {linenos=table, filename="example.slurm"}
#!/bin/bash
#SBATCH -J example
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 02:00:00
#SBATCH -o %x-%j.out
#SBATCH -e %x-%j.err

echo "Job $SLURM_JOB_ID on $SLURM_JOB_NODELIST"
echo "Submitted from $SLURM_SUBMIT_DIR"
echo "CPUs: $SLURM_CPUS_PER_TASK"

module load gnu12
which gcc

WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"

# Copy inputs from $DATA_DIR into $WORKDIR, run, copy keepers back.
srun hostname
```

Submit it from the login node:

```console
$ sbatch example.slurm
Submitted batch job 238
```

What each header line does:

| Directive | Meaning |
| --------- | ------- |
| `-J example` | Job name (shows up in `squeue`; `%x` in filenames). |
| `-c 8` | Eight CPUs for this task. Use this for multithreaded programs. |
| `--mem-per-cpu=4G` | RAM **per CPU**. Here: 8 × 4 GB = 32 GB total. The cluster default is already ~4 GB per CPU; writing it down makes the request obvious. |
| `-t 02:00:00` | Maximum wall time (2 hours). Format: `HH:MM:SS` or `days-HH:MM:SS`. Maximum is 14 days. |
| `-o %x-%j.out` | Standard output file. `%j` is the job ID, `%x` is the job name. |
| `-e %x-%j.err` | Standard error in a separate file. Omit `-e` to keep both streams in `-o`. |

Useful extras (not required):

| Directive | Meaning |
| --------- | ------- |
| `--gpus=l4:1` | One GPU of type `l4` (see [GPUs](#gpus)). |
| `--constraint=cpu-amd` | Only nodes tagged with that [feature](#features). |
| `--mail-user=` / `--mail-type=` | [Email]({{< relref "slurm#notification-system" >}}). |

## Launching the program with `srun` {#srun}

In a **batch** script the commands already run on the allocated compute node.
`echo`, `module load`, `cp`, and `cd` can stay as ordinary shell commands.

The **application itself** should be started with `srun`.
That creates a Slurm *job step*: the scheduler then knows which process is the workload, can bind it to the reserved cores, and can run MPI correctly.

```bash
module load gnu12 openmpi4
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
srun ./my_program arg1 arg2
```

For MPI, `srun` replaces `mpirun` / `mpiexec` on this cluster:

```bash
#SBATCH -n 32
#SBATCH --mem-per-cpu=4G
module load gnu12 openmpi4
srun --cpu-bind=cores ./mpi_app
```

`-n` is the number of MPI ranks (tasks).
Keep `n` (or `n × cpus-per-task`) within **one** node (at most 128 cores).

> [!NOTE]
> Do not type `srun -c 8 ./app` on the login node as a substitute for `sbatch`.
> That form **creates a new allocation** and is an interactive shortcut; it belongs on [Interactive jobs]({{< relref "job-interactive" >}}).
> Inside a script that you already submitted with `sbatch`, write `srun ./app` (the resources are already reserved).

If the program must run in the background so the script can catch a time-limit signal, the pattern is `srun ... &` then `wait`, as in the GROMACS example in [Getting Started]({{< relref "getting-started" >}}).

## Resource options in more detail

### CPUs: `-c` vs `-n`

- `#SBATCH -c 8` (same as `--cpus-per-task=8`): one task, eight cores. Typical for OpenMP or a program with a `--threads` flag.
- `#SBATCH -n 16` (same as `--ntasks=16`): sixteen tasks, one core each unless you also set `-c`. Typical for MPI.

You can combine them (`-n 4 -c 8` is 4 ranks × 8 threads = 32 cores), still on a **single** node.

Tell the program about the reservation:

```bash
#SBATCH -c 8
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
```

If you do not, many codes will see every core on the node and oversubscribe it.

### Memory: `--mem-per-cpu`

Prefer **memory per CPU**, which scales when you change `-c`:

```bash
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G    # 32 GB total
```

```bash
#SBATCH -c 8
#SBATCH --mem-per-cpu=8G    # 64 GB total, for a memory-heavy run
```

`--mem=32G` is the older “total RAM” form.
It does not change when you edit `-c`, so it is easier to get wrong.
Use `--mem-per-cpu` unless you have a reason not to.

If the job is killed with `OUT_OF_MEMORY`, raise `--mem-per-cpu` or reduce the problem size.
`sacct` reports `MaxRSS` after the job ends ([Job management]({{< relref "job-management" >}})).

### Time, output files, working directory

`-t` is a **limit**, not an estimate that the job will take that long.
The script’s current directory at start is the directory where you ran `sbatch` (`$SLURM_SUBMIT_DIR`).
That is usually HOME, which is slow.
Create a directory on scratch, `cd` there, then copy results back (see the example at the top of this page).

## Features and `--constraint` {#features}

Each node carries **tags** (features): CPU vendor, CPU model, GPU family, and so on.
They let you request a *kind* of machine without naming `rose` or `sina`.

Most jobs should **not** set `--constraint`.
Slurm already picks a node that has enough CPUs, memory, and GPUs.
Add a constraint only when the program requires it (a binary that only runs on AMD, or a GPU model).

List what is installed:

```console
$ sinfo -N -o '%N %c %m %G %f'
NODELIST   CPUS MEMORY    GRES                           AVAIL_FEATURES
nc01         32 257574    (null)                         cpu-intel,cpu-cascade-lake,cpu-xeon,xeon-silver-4216
nc05         64 515400    (null)                         cpu-amd,cpu-zen4,cpu-epyc,epyc-9534,mem512
maria       128 322215    (null)                         cpu-amd,cpu-zen3,cpu-epyc,epyc-7713,mem320
rose        128 483481    (null)                         cpu-amd,cpu-zen3,cpu-epyc,epyc-7713,mem512
sina         40 514469    gpu:l4:2                       cpu-intel,...,gpu,gpu-l4
vision       24  57344    gpu:2080ti:2,gpu:3090:1        cpu-intel,...,gpu-2080ti,gpu-3090
wc01         16 128439    gpu:2080ti:1,gpu:2080super:1   cpu-intel,...,gpu-2080ti,gpu-2080super
```

(The listing is shortened; run the command for the live list.)

Examples:

```bash
# Only AMD CPUs (nc05, maria, rose)
#SBATCH --constraint=cpu-amd

# The large EPYC 7713 nodes (maria, rose)
#SBATCH --constraint=epyc-7713
```

`--constraint` is an extra filter on top of CPUs/memory/GPUs.
Impossible combinations never start, for example `--gpus=l4:1` together with `--constraint=epyc-7713` (`sina` has the L4, the EPYC nodes do not).
The error looks like `Requested node configuration is not available`.

For GPUs, prefer `--gpus=` (below) over a GPU feature tag.
`--gpus=l4:1` already implies a node that has that device.

## GPUs {#gpus}

Only three nodes have GPUs:

| Node | What Slurm calls the devices |
| ---- | ---------------------------- |
| sina | `gpu:l4:2` |
| vision | `gpu:2080ti:2,gpu:3090:1` |
| wc01 | `gpu:2080ti:1,gpu:2080super:1` |

```bash
#SBATCH --gpus=l4:1          # one NVIDIA L4 (sina)
#SBATCH --gpus=2080ti:1      # one RTX 2080 Ti (vision or wc01)
#SBATCH --gpus=1             # whatever is free (can be much slower)
```

Load CUDA as well as the application module (`module load cuda` is often pulled in by the app module).
A complete GPU batch script, including scratch and a time-limit trap, is the GROMACS example in [Getting Started]({{< relref "getting-started" >}}).

## Environment variables

Slurm sets these in the script (among others):

| Variable | Content |
| -------- | ------- |
| `$SLURM_JOB_ID` | Numeric job id |
| `$SLURM_JOB_NAME` | Name from `-J` |
| `$SLURM_CPUS_PER_TASK` | CPUs from `-c` |
| `$SLURM_NPROCS` / `$SLURM_NTASKS` | Task count from `-n` |
| `$SLURM_JOB_NODELIST` | Compute node hostname |
| `$SLURM_SUBMIT_DIR` | Directory where you ran `sbatch` |
| `$SLURM_GPUS_ON_NODE` | GPUs on the node, if any |

`$DATA_DIR` and `$SCRATCH_DIR` come from the cluster login environment, not from Slurm; they are still set in batch jobs.

## Complete examples

### Multithreaded CPU program

```bash {filename="threads.slurm"}
#!/bin/bash
#SBATCH -J threads
#SBATCH -c 16
#SBATCH --mem-per-cpu=4G
#SBATCH -t 08:00:00
#SBATCH -o %x-%j.out

module load gnu12
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$DATA_DIR"/inputs/data.bin .

srun ./my_openmp_app data.bin

cp -a results/ "$DATA_DIR"/results/"$SLURM_JOB_ID"/
```

Application-specific scripts (quality control, aligners, and so on) live under [Software]({{< relref "software" >}}).

### MPI on one node

```bash {filename="mpi.slurm"}
#!/bin/bash
#SBATCH -J mpi-app
#SBATCH -n 32
#SBATCH --mem-per-cpu=4G
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load gnu12 openmpi4
srun --cpu-bind=cores ./mpi_app
```

Keep `-n` ≤ cores on the node you will land on (32 fits on `nc01`–`nc04`; 128 is the ceiling on `rose`/`maria`).

### GPU

```bash {filename="gpu.slurm"}
#!/bin/bash
#SBATCH -J gpu-app
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH --gpus=2080ti:1
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load cuda
srun ./my_gpu_app
```

If performance depends on the GPU model, do not use `--gpus=1`.
