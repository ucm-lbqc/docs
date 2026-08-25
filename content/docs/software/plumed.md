---
title: PLUMED
---

[PLUMED](https://www.plumed.org/) adds enhanced sampling, free-energy methods, and analysis on top of MD engines.

| | |
| --- | --- |
| Module | `mm/plumed/2.10a` |
| Depends on | `gnu12`, `openmpi4`, OpenBLAS, ScaLAPACK, GSL, FFTW, Boost, `lang/python` |
| Commands | `plumed`, `plumed-patch` |
| Kernel | `$PLUMED_KERNEL` |

This cluster’s [DFTB+]({{< relref "dftbplus" >}}) module already depends on this PLUMED build.
GROMACS/NAMD/LAMMPS on the cluster are **not** automatically PLUMED-patched; use `plumed` for analysis (`driver`) or ask for a patched MD build if you need it during the simulation.

## Load

```console
$ module load mm/plumed
```

## Example (analysis)

```bash {filename="plumed.slurm"}
#!/bin/bash
#SBATCH -J plumed
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 04:00:00
#SBATCH -o %x-%j.out

module load mm/plumed
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
srun plumed driver --mf_xtc "$DATA_DIR/traj.xtc" --plumed plumed.dat
```

`plumed.dat` is your PLUMED input (CVs, PRINT, etc.).
See the [PLUMED documentation](https://www.plumed.org/doc).
