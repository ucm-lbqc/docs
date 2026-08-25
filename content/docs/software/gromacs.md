---
title: GROMACS
---

[GROMACS](https://www.gromacs.org/) is a molecular-dynamics and analysis suite.
The cluster build is **CUDA-enabled**.

| | |
| --- | --- |
| Module | `md/gromacs/2024.1+cuda` |
| Depends on | `gnu12`, `openmpi4`, `cuda` (loaded automatically) |
| Commands | `gmx`, `gmx_mpi` |

## Load

```console
$ module load md/gromacs
```

## Example (GPU)

A complete production script (scratch, mail, time-limit trap) is in [Getting Started]({{< relref "getting-started" >}}).
A shorter GPU run:

```bash {filename="gromacs.slurm"}
#!/bin/bash
#SBATCH -J gmx
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH --gpus=2080ti:1
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load md/gromacs
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/system.tpr .

srun gmx mdrun -nt "$SLURM_CPUS_PER_TASK" -deffnm system
cp -a system.* "$SLURM_SUBMIT_DIR"/
```

`-nt` must not exceed the CPUs you reserved.
`--gpus=1` takes whichever GPU is free; name a type if performance matters.

`gmx_mpi` is for MPI ranks (`#SBATCH -n`, still **one** node).
Most GPU runs use thread-MPI `gmx` as above.

Analysis (`gmx rms`, `gmx energy`, …) is lighter but still belongs in a job if it reads large trajectories.
