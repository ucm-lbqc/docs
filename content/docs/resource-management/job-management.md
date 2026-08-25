---
title: Job Management
next: job-interactive
weight: 703
---

Once a job is submitted, you inspect it, read its output, and cancel it if needed.
[Getting Started]({{< relref "getting-started" >}}) already shows a first `squeue`.
This page is the same commands with more output to read and more options.

## squeue — jobs that are waiting or running

```console
$ squeue --me
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
               238    normal example   fadasme  R       2:35      1 rose
               241    normal example   fadasme PD       0:00      1 (Resources)
```

| Column | Meaning |
| ------ | ------- |
| `JOBID` | Number you pass to `scancel`, `scontrol`, `sacct`. |
| `PARTITION` | Always `normal` on this cluster. |
| `NAME` | From `#SBATCH -J` (truncated). |
| `ST` | State: `R` running, `PD` pending, `CG` completing. |
| `TIME` | How long it has been running (or waiting, for `PD`). |
| `NODELIST(REASON)` | Hostname if running; a reason in parentheses if pending. |

One job:

```console
$ squeue -j 238
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
               238    normal example   fadasme  R       2:35      1 rose
```

`squeue` with no arguments lists **everyone’s** jobs. Prefer `squeue --me`.

### Pending reasons {#pending}

When `ST` is `PD`, the last column explains why the job has not started.

```console
$ squeue --me
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
               241    normal example   fadasme PD       1:23      1 (Resources)
               242    normal bigmem    fadasme PD       0:40      1 (Priority)
```

| Reason | Meaning |
| ------ | ------- |
| `(Resources)` | No node currently has enough free CPUs, memory, or GPUs. The job will start when something finishes, or you can resubmit with a smaller request. |
| `(Priority)` | Other jobs are ahead. Wait, or ask for less so you fit a smaller node. |
| `(QOSMax…)` | A limit was hit. Rare here; there is no extra QoS. |
| `CPU count per node can not be satisfied` | You asked for more cores than any **single** node has (max 128). Remember: one node per job. |
| `Requested node configuration is not available` | The combination cannot exist (GPU type vs CPU `--constraint`, or a GPU count that no node has). |

```console
$ sbatch too-big.slurm
sbatch: error: CPU count per node can not be satisfied
sbatch: error: Batch job submission failed: Requested node configuration is not available
```

More context for a single job:

```console
$ scontrol show job 241
JobId=241 JobName=example
   UserId=fadasme(1000) GroupId=fadasme(1000)
   Priority=4294901758 Nice=0
   JobState=PENDING Reason=Resources Dependency=(null)
   ReqNodeList=(null) ExclNodeList=(null)
   NumNodes=1 NumCPUs=8 NumTasks=1 CPUs/Task=8
   MinMemoryCPU=4G
   TimeLimit=02:00:00
   ...
```

Look at `JobState`, `Reason`, `NumCPUs`, `MinMemoryCPU`, and `TimeLimit` if the job sits in `PD` longer than you expect.

## sinfo — are the nodes up?

```console
$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
normal*      up 14-00:00:00      2   mix  nc01,sina
normal*      up 14-00:00:00      1 alloc rose
normal*      up 14-00:00:00      7  idle nc0[2-5],maria,vision,wc01
```

| State | Meaning |
| ----- | ------- |
| `idle` | Nothing running; free for new jobs. |
| `mix` | Some CPUs in use, some free. |
| `alloc` | Fully allocated. |
| `drain` / `down` | Not accepting jobs. `%E` shows why. |

Per node, with memory, GPUs, and features:

```console
$ sinfo -N -o '%N %c %m %G %T %E'
NODELIST   CPUS MEMORY    GRES                    STATE  REASON
nc01         32 257574    (null)                  mix    none
rose        128 483481    (null)                  alloc  none
sina         40 514469    gpu:l4:2                mix    none
```

## sacct — jobs that already finished

`squeue` only sees the queue.
`sacct` is the accounting database (completed jobs, and running ones):

```console
$ sacct -j 238 --format=JobID,JobName,State,Elapsed,MaxRSS,ExitCode
JobID           JobName      State    Elapsed     MaxRSS ExitCode
------------ ---------- ---------- ---------- ---------- --------
238             example  COMPLETED   00:04:12
238.batch         batch  COMPLETED   00:04:12    512348K      0:0
238.0              hostname  COMPLETED   00:00:01      1234K      0:0
```

| State | Meaning |
| ----- | ------- |
| `COMPLETED` | Script exited 0. |
| `FAILED` | Non-zero exit. Read the `.err` / `.out` file. |
| `TIMEOUT` | Hit `-t`. |
| `CANCELLED` | `scancel`, or the user logged out of an allocation. |
| `OUT_OF_MEMORY` | Exceeded the reserved RAM. Raise `--mem-per-cpu`. |

Your jobs since a date:

```console
$ sacct --me --starttime=2026-08-01
```

The `.batch` line is the script itself; `.0`, `.1`, … are `srun` steps.

## scancel — stop a job

```console
$ scancel 238
$ scancel -n example
$ scancel --me
```

`scancel --me` cancels **all** of your jobs (pending and running). There is no undo.

## Output files

By default Slurm writes `slurm-<jobid>.out` in the directory where you ran `sbatch`.
If the script set `-o %x-%j.out`, look for `example-238.out` instead.

```console
$ cat example-238.out
Job 238 on rose
Submitted from /home/fadasme/jobs
CPUs: 8
```

If the file is empty while `ST` is `R`, the program may still be buffering; wait, or check `scontrol show job` for `StdOut=`.

## SSH to a compute node

Slurm PAM blocks SSH unless you already have a **running** job on that node.

For a shell, use an [interactive job]({{< relref "job-interactive" >}}) rather than SSH.
If a batch job is already running, `ssh <node>` from `lbqc` may work for debugging **that** job only.
Do not start extra compute there.
