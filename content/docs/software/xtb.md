---
title: xTB
---

[xTB](https://xtb-docs.readthedocs.io) implements the GFNn-xTB family of semiempirical tight-binding methods (and related composite schemes).

| | |
| --- | --- |
| Modules | `qm/xtb/6.7.1` (newer), `qm/xtb/6.7.0` |
| Commands | `xtb`, `cpx` |
| Parameters | `$XTBPATH` |

The module sets `OMP_NUM_THREADS` from `$SLURM_CPUS_PER_TASK` (or `1` if you are not in a job), `OMP_STACKSIZE=4G`, and raises the stack size.
**Load it inside the job** so the thread count matches `-c`.

## Load

```console
$ module load qm/xtb
```

## Example

```bash {filename="xtb.slurm"}
#!/bin/bash
#SBATCH -J xtb
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 08:00:00
#SBATCH -o %x-%j.out

module load qm/xtb
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/mol.xyz .

srun xtb mol.xyz --opt --chrg 0
cp -a xtbopt.xyz "$SLURM_SUBMIT_DIR"/
```

`--opt` is a geometry optimization; see `xtb --help` for single points, frequencies, and GBSA/ALPB solvents.
Convert structures with [Open Babel]({{< relref "openbabel" >}}) if you need XYZ.
