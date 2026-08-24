---
title: Resource Management
prev: software
next: slurm
weight: 700
sidebar:
  open: true
---

Compute nodes are scheduled with [Slurm](https://slurm.schedmd.com/).
You never log in to a node to “just run” a production job: you **submit** work, and Slurm picks a node that matches the request.

{{< cards >}}
  {{< card link="slurm" title="SLURM" subtitle="Partitions, time limits, mail, scheduling" >}}
  {{< card link="job-submission" title="Job submission" subtitle="sbatch, salloc, srun, GPUs" >}}
  {{< card link="job-management" title="Job management" subtitle="squeue, scancel, sacct" >}}
  {{< card link="job-interactive" title="Interactive jobs" subtitle="salloc and srun --pty" >}}
{{< /cards >}}

A first walkthrough is in [Getting Started]({{< relref "getting-started" >}}).
Cluster-wide habits: [Best practices]({{< relref "best-practices" >}}).
When a job will not start: [Troubleshooting]({{< relref "troubleshooting" >}}).
