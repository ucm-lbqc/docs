---
title: Best practices
prev: job-interactive
next: troubleshooting
weight: 800
---

Short rules that keep the cluster usable for everyone.

## Login node

The login node is for SSH, editing, `sbatch`/`squeue`, and small file copies.
Compiling STAR-sized projects, `fasterq-dump`, FastQC over many samples, or `gmx mdrun` here can get your account disabled.

## One node per job

The `normal` partition has `MaxNodes=1`.
Do not request MPI across several nodes; it will not be scheduled.
Use several CPUs on **one** node (`#SBATCH -c` or `--ntasks` on that node) or submit independent jobs.

## Memory

Default memory is about **4 GB per CPU** (`DefMemPerCPU`).
STAR and TElocal need much more: set `#SBATCH --mem=32G` (or higher) explicitly.
If you omit `--mem` and take 16 CPUs, you already get a large allocation; do not also oversubscribe the node with OpenMP threads beyond `$SLURM_CPUS_PER_TASK`.

## Scratch vs data

- Stage big I/O in `$SCRATCH_DIR`.
- Keep genomes, indices, and final results in `$DATA_DIR`.
- Copy back **before** scratch is wiped (one week).
- RAID is not a backup ([Storage]({{< relref "storage" >}})).

## Modules belong in the job script

`module load` in an interactive SSH session does not apply to `sbatch` jobs.
Load modules in the script (or in `salloc` after the allocation starts).

## Threads and Java

Pin thread counts to the allocation:

```bash
#SBATCH -c 8
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
```

For Trimmomatic, also set `-threads "$SLURM_CPUS_PER_TASK"` and `MALLOC_ARENA_MAX=2` ([Trimmomatic]({{< relref "trimmomatic" >}})).

## GPUs

Ask for a type if the code cares (`--gpus=l4:1` vs `--gpus=2080ti:1`).
`--gpus=1` takes whichever GPU is free and may be much slower.
GPU nodes are `wc01`, `sina`, and `vision` ([Cluster]({{< relref "cluster" >}})).

## Time limits

Set `-t` to a realistic upper bound.
The partition allows up to **14 days**.
Shorter limits start sooner when the cluster is busy.
For graceful shutdown, see the GROMACS example in [Getting Started]({{< relref "getting-started" >}}) (`--signal=B:USR1`).

## Bind MPI ranks to cores

On this cluster HT is disabled; bind to cores:

```bash
srun --cpu-bind=cores ./my_mpi_app
```

VASP in particular is slow if ranks float across cores.
