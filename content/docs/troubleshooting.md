---
title: Troubleshooting
prev: best-practices
weight: 900
---

## Job stuck in `PD` (pending)

`squeue` shows a reason in the last column.

| Reason | Meaning |
| ------ | ------- |
| `(Resources)` | No node currently matches CPUs, memory, GPU, or features. Wait, or lower the request. |
| `(Priority)` | Other jobs are ahead. |
| `(QOSMax... )` | A limit was hit (rare here; there is no extra QoS). |
| `CPU count per node can not be satisfied` | You asked for more cores than any **single** node has (max 128 on rose/maria). Remember **one node per job**. |
| `Requested node configuration is not available` | Impossible combo (e.g. `--gpus=l4:1` and `--constraint=epyc-7713`). |

Inspect with `scontrol show job <id>` and `sinfo -N -l`.

## `module: command not found` / empty `PATH` in a job

The job script did not load the module.
Login-node `module load` does not carry over.
Put `module load ...` in the script.

## `command not found` after `module load`

Wrong name (`fastqc` vs `FastQC`), or you loaded a conda-based module and then ran a different `python`.
Use `which fastqc` / `which TElocal` in the script.

## `No space left on device`

HOME or `$DATA_DIR` is full, or scratch on that node is full.
Delete or move data ([Storage]({{< relref "storage" >}})).
Do not fill NFS with STAR temp files.

## `Illegal instruction` (STAR or other binaries)

You ran a binary built for a newer CPU on an older node (or the reverse).
Use the cluster module, or rebuild without aggressive `-march=native`.
STAR here is built with AVX2, which matches the current nodes.

## Cannot SSH to a compute node

Direct SSH is allowed only while you have a **running job** on that node (Slurm PAM).
Use `srun --pty bash` or `salloc` instead ([Interactive jobs]({{< relref "job-interactive" >}})).

## Java / Trimmomatic dies immediately with a huge virtual memory error

Set `MALLOC_ARENA_MAX=2` and `-threads "$SLURM_CPUS_PER_TASK"` ([Trimmomatic]({{< relref "trimmomatic" >}})).

## Still stuck

Email [fadasme@ucm.cl](mailto:fadasme@ucm.cl) with the **job ID**, the script, and `scontrol show job <id>` output.
