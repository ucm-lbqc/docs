---
title: Schrödinger
---

The [Schrödinger Suite](https://www.schrodinger.com/) is a commercial platform for molecular modeling, drug discovery, and materials (Maestro, Glide, Prime, Desmond inside the suite, and others).

| | |
| --- | --- |
| Modules | `mm/schrodinger/2026-1` (newer), `mm/schrodinger/2023-4` |
| Home | `$SCHRODINGER` |
| Scratch | `$SCHRODINGER_TMPDIR` (set to `$SCRATCH_DIR`) |

Licenses come from the lab’s Schrödinger license server. Use the suite only under that academic license; do not copy the tree off the cluster.

Do **not** load this together with [Desmond]({{< relref "desmond" >}}) (`md/desmond`): both set `$SCHRODINGER`.

## Load

```console
$ module load mm/schrodinger
```

Pick a version if you need the older release:

```console
$ module load mm/schrodinger/2023-4
```

Check that the license server is visible (from a compute node):

```console
$ $SCHRODINGER/run lictool status
```

## Batch jobs

Maestro is a GUI: use it on a workstation with a display, not on the login node.
Command-line tools go through `$SCHRODINGER/run`.

```bash {filename="schrodinger.slurm"}
#!/bin/bash
#SBATCH -J schrod
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load mm/schrodinger
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/job.inp .

srun $SCHRODINGER/run <workflow> job.inp
```

Replace `<workflow>` with the program you need (`glide`, `prime`, and so on).
GPU-enabled Schrödinger jobs also need `#SBATCH --gpus=…`.
Temporary files already go to `$SCRATCH_DIR` via the module.
