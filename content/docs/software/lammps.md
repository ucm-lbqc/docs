---
title: LAMMPS
---

[LAMMPS](https://www.lammps.org/) (Large-scale Atomic/Molecular Massively Parallel Simulator) is a classical MD code, widely used for materials.

| | |
| --- | --- |
| Modules | `md/lammps/2024-08` (CPU), `md/lammps/2024-08+cuda` (GPU) |
| Depends on | `gnu12`, `openmpi4`, OpenBLAS, ScaLAPACK; CUDA on the GPU build |
| Command | `lmp` |
| Potentials | `$LAMMPS_POTENTIALS` |

## Load

```console
$ module load md/lammps
```

GPU build:

```console
$ module load md/lammps/2024-08+cuda
```

## Example

Input files are LAMMPS scripts (often `.in`). Launch with `srun` so MPI ranks bind to cores.

```bash {filename="lammps.slurm"}
#!/bin/bash
#SBATCH -J lammps
#SBATCH -n 32
#SBATCH --mem-per-cpu=4G
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load md/lammps
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/in.system .

srun --cpu-bind=cores lmp -in in.system
```

Keep `-n` ≤ cores on one node (max 128).
For the CUDA package, add `#SBATCH --gpus=…`, load `md/lammps/2024-08+cuda`, and follow the GPU settings in your input (`package gpu` / `suffix gpu`).
