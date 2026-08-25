---
title: VMD
---

[VMD](https://www.ks.uiuc.edu/Research/vmd/) (Visual Molecular Dynamics) displays, animates, and analyzes biomolecular systems.

| | |
| --- | --- |
| Module | `mm/vmd/1.9.4a57` |
| Command | `vmd` |
| Home | `$VMDDIR` |

The OpenGL GUI needs a local display. It is **not** supported on the login node or on headless compute nodes in a useful way.
For cluster jobs, run VMD in **text** mode with a Tcl script.

## Load

```console
$ module load mm/vmd
```

## Example (batch analysis)

```bash {filename="vmd.slurm"}
#!/bin/bash
#SBATCH -J vmd
#SBATCH -c 4
#SBATCH --mem-per-cpu=4G
#SBATCH -t 02:00:00
#SBATCH -o %x-%j.out

module load mm/vmd
srun vmd -dispdev text -e "$SLURM_SUBMIT_DIR"/analyze.tcl
```

`analyze.tcl` is your Tcl script (`mol load`, `measure`, writes of data files).
Keep large trajectories on `$DATA_DIR` or scratch; do not stream them through HOME.

Interactive viewing: copy a subset of the trajectory to your laptop, or use a workstation with graphics.
NAMD users often pair VMD with [NAMD]({{< relref "namd" >}}).
