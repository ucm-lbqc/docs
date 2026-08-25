---
title: Troubleshooting
prev: best-practices
weight: 900
---

Problems you will hit sooner or later, and what they usually mean.
For the first walkthrough of a successful job, stay on [Getting Started]({{< relref "getting-started" >}}).

## Job stuck in `PD` (pending)

`squeue` prints a reason in the last column.
A full list with example output is in [Job management]({{< relref "job-management#pending" >}}). The short version:

| Reason | What to do |
| ------ | ---------- |
| `(Resources)` | No free node currently has the CPUs, memory, or GPU you asked for. Wait, or lower the request. |
| `(Priority)` | Other jobs are ahead in the queue. Wait, or ask for less so you fit a smaller node. |
| `CPU count per node can not be satisfied` | You asked for more cores than any **single** node has (maximum 128). This cluster does not run one job on two machines. |
| `Requested node configuration is not available` | The combination cannot exist (for example a GPU that only `sina` has, plus a CPU feature that `sina` does not have). |

Inspect the job with `scontrol show job <id>` and the nodes with `sinfo -N -l`.

## `module: command not found` inside a job

The job script did not load the module, or it started from a shell that never sourced the module system.
Loading a module in an interactive SSH session on `lbqc` does **not** carry over to `sbatch`.
Put `module load ...` in the script itself.

## `command not found` after `module load`

The command name is not what you think (modules are not always named after the binary), or you loaded one Python environment and then called a different `python`.
Add `which <command>` and `module list` in the script and check the output file.

## `No space left on device`

HOME, `$DATA_DIR`, or the scratch disk on that node is full.
Delete or move data ([Storage]({{< relref "storage" >}})).
Do not write large temporary files to NFS (HOME or `$DATA_DIR`); use `$SCRATCH_DIR`.

## `Illegal instruction`

The program was compiled for a CPU instruction set that the node does not implement (or the reverse: an old binary on a new node is usually fine; a new binary on an old node is not).
Use the cluster module if one exists, or rebuild without flags such as `-march=native` unless you also constrain the job to that CPU family ([Job submission]({{< relref "job-submission#features" >}})).

## Cannot SSH to a compute node

Direct SSH is allowed only while you already have a **running** job on that node.
To get a shell for testing, use an [interactive job]({{< relref "job-interactive" >}}) (`salloc` or `srun --pty`).

## Job dies immediately with a huge virtual-memory error (often Java)

Some runtimes, especially the Java virtual machine, reserve a large block of virtual memory **per thread**.
If you requested 8 CPUs but the program starts 64 threads, the job can be killed before it does any work.

Cap threads to the allocation (`$SLURM_CPUS_PER_TASK` and the program’s own thread flag) and, for Java, set:

```bash
export MALLOC_ARENA_MAX=2
```

The [Trimmomatic]({{< relref "trimmomatic" >}}) guide shows a complete example.

## Still stuck

Email [fadasme@ucm.cl](mailto:fadasme@ucm.cl) with the **job ID**, the script, and the output of `scontrol show job <id>`.
