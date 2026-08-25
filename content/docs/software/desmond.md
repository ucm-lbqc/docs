---
title: Desmond
---

[Desmond](https://www.deshawresearch.com/resources.html) is D. E. Shaw Research’s GPU molecular-dynamics engine.
Academic and not-for-profit use is free of charge; the cluster install uses that academic license.

| | |
| --- | --- |
| Module | `md/desmond/2023-4` |
| Commands | `desmond`, `maestro`, `$SCHRODINGER/run` |
| Home | `$SCHRODINGER` |

Desmond is GPU-oriented. Request a GPU ([Job submission]({{< relref "job-submission#gpus" >}})).

This module is the **Desmond** academic bundle, not the full [Schrödinger Suite]({{< relref "schrodinger" >}}) (`mm/schrodinger`).
Do not load both at once: both set `$SCHRODINGER`.

## Load

```console
$ module load md/desmond
```

## Example

Prepare the system in Maestro (on a workstation with a display) or with `$SCHRODINGER/run`.
Submit the MD on the cluster:

```bash {filename="desmond.slurm"}
#!/bin/bash
#SBATCH -J desmond
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH --gpus=2080ti:1
#SBATCH -t 24:00:00
#SBATCH -o %x-%j.out

module load md/desmond
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/*.cms "$SLURM_SUBMIT_DIR"/*.cfg . 2>/dev/null || true

srun $SCHRODINGER/desmond -WAIT -in job.cfg
```

Adjust `-in` to your Desmond config name.
Copy keepers back to `$DATA_DIR` or the submit directory when the job finishes.

Maestro’s GUI needs a local display; it is not meant for the login node or a headless compute node.
