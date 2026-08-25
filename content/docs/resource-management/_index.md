---
title: Resource Management
prev: software
next: slurm
weight: 700
sidebar:
  open: true
---

Compute nodes are scheduled with [Slurm](https://slurm.schedmd.com/).
You never log in to a node to “just run” a production job: you **describe** the resources, **submit** the work, and Slurm picks a machine that matches.

If you have never submitted a job, start with [Getting Started]({{< relref "getting-started" >}}).
This section is the reference: more options, more explanation, and complete examples.

{{< cards >}}
  {{< card link="slurm" title="SLURM" subtitle="What the scheduler does: partitions, time, mail" >}}
  {{< card link="job-submission" title="Job submission" subtitle="sbatch scripts, resources, srun inside a job" >}}
  {{< card link="job-management" title="Job management" subtitle="squeue, scancel, sacct, sinfo" >}}
  {{< card link="job-interactive" title="Interactive jobs" subtitle="salloc and srun --pty on a compute node" >}}
{{< /cards >}}

Cluster-wide habits: [Best practices]({{< relref "best-practices" >}}).
When a job will not start: [Troubleshooting]({{< relref "troubleshooting" >}}).
