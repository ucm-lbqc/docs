---
title: Best practices
prev: job-interactive
next: troubleshooting
weight: 800
---

These habits keep the cluster usable for everyone.
They are the same ideas as in [Getting Started]({{< relref "getting-started" >}}), written as rules you can check before you submit.

## Stay off the login node for heavy work

The login node (`lbqc`) is a shared front door: SSH, editors, `sbatch`, `squeue`, and small file copies.

Do **not** run there:

- Analyses or simulations (anything that would take minutes on a laptop, or that uses many CPU cores)
- Compiling large packages
- Downloading or unpacking large datasets
- Anything that saturates disk or network for a long time

Those tasks belong in a Slurm job on a compute node.
Misuse of the login node can get the account disabled.

## One node per job

The default partition allows **one machine per job**.
You can use many cores on that machine, but you cannot split one job across `rose` and `maria`.

If you need more work done, submit several independent jobs (one sample, one simulation, one parameter set each), or use the cores of a single large node (up to 128).

## Ask for memory per CPU

By default the cluster gives about **4 GB of RAM per reserved CPU**.
You almost always want to set this explicitly with `--mem-per-cpu` so it is visible in the script:

```bash
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
```

That example reserves 8 CPUs and 32 GB in total (8 × 4 GB).
Increase `--mem-per-cpu` if the program is memory-hungry (genome indexing, large molecular systems, big matrices).
If you omit it, you still get the 4 GB-per-CPU default, but the script is harder to read and to debug.

Do not start more threads than the CPUs you reserved: see below.

## Work on scratch, keep results on data

| Put here | Typical contents |
| -------- | ---------------- |
| `$SCRATCH_DIR` | Temporary files, working directories, anything the job only needs while it runs |
| `$DATA_DIR` | Inputs you will reuse, and outputs you must keep |
| `$HOME` | Scripts and configuration, not datasets |

Copy keepers **before** the job ends: scratch is wiped after a week of inactivity.
RAID on the NAS is not a backup ([Storage]({{< relref "storage" >}})).

## Load modules inside the job

`module load` in your SSH session does **not** apply to a job you submit later with `sbatch`.
Put every `module load` in the job script (or after an interactive allocation has started).

## Tell the program how many CPUs it may use

You reserved cores with `#SBATCH -c`. The program does not guess that number by itself.
Many tools will try to use every core they see on the node (for example 128 on `rose`) even if you only asked for 8.

```bash
#SBATCH -c 8
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
```

`$SLURM_CPUS_PER_TASK` is set by Slurm to the value of `-c`.
Pass the same number to the application if it has its own `--threads` / `-nt` flag.

Some Java programs also need `export MALLOC_ARENA_MAX=2` so they do not reserve a huge amount of virtual memory per thread. See [Troubleshooting]({{< relref "troubleshooting" >}}).

## Request a GPU type when it matters

`--gpus=1` takes whichever GPU is free.
A data-center GPU and a gaming GPU are not interchangeable: the job can be much slower, or the software may refuse to start.

If you care about the model, ask for it (`--gpus=l4:1` or `--gpus=2080ti:1`).
GPUs exist only on `wc01`, `sina`, and `vision` ([Cluster]({{< relref "cluster" >}})).

## Set a realistic time limit

`-t` is the **maximum** the job is allowed to run, not a prediction that it will take that long.
The partition allows up to **14 days**.
Shorter limits usually start sooner when the cluster is busy, and they free the node if the job hangs.

If the program can save a checkpoint, you can catch the time-limit signal and copy files back; the GROMACS script in [Getting Started]({{< relref "getting-started" >}}) shows the pattern (`--signal=B:USR1`).

## Pin MPI processes to cores

Hyper-threading is disabled on this cluster: each reserved CPU is a real core.
If you run an MPI program, bind ranks to cores so they do not migrate:

```bash
srun --cpu-bind=cores ./my_mpi_app
```

Without binding, some codes spend a lot of time moving between cores and run much slower.
