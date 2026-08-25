---
title: JDFTx
---

[JDFTx](https://jdftx.org/) is a plane-wave density-functional theory code (joint DFT, solvation, and related methods).

| | |
| --- | --- |
| Module | `qm/jdftx/1.7.0@12396ea` |
| Depends on | `gnu12`, `openmpi4`, FFTW, GSL, OpenBLAS, ScaLAPACK |
| Commands | `jdftx`, `phonon`, `wannier` |
| Home | `$JDFTX_DIR` |

The version includes a git commit (`@12396ea`).

## Load

```console
$ module load qm/jdftx
```

## Example

```bash {filename="jdftx.slurm"}
#!/bin/bash
#SBATCH -J jdftx
#SBATCH -n 16
#SBATCH --mem-per-cpu=4G
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load qm/jdftx
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/in.jdftx .

srun --cpu-bind=cores jdftx -i in.jdftx
```

Keep `-n` on a single node.
Input syntax is documented at [jdftx.org](https://jdftx.org/).
Copy `out` / charge-density files back to `$DATA_DIR` if you need them.
