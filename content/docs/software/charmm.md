---
title: CHARMM
---

[CHARMM](https://www.charmm.org/) (Chemistry at HARvard Macromolecular Mechanics) is a molecular simulation program.

| | |
| --- | --- |
| Modules | `md/charmm/48b2` (CPU, Intel + MPI), `md/charmm/48b2+cuda` (GPU) |
| Command | `charmm` |

Licensed software. Use it only under the lab’s CHARMM license; do not copy the binaries.

## Load

```console
$ module load md/charmm
```

That loads the default variant (`48b2` unless Lmod marks otherwise).
For a GPU run, load the CUDA build explicitly:

```console
$ module load md/charmm/48b2+cuda
```

## Example

CHARMM reads an input script (often `.inp`). Run on a compute node; keep large trajectories on scratch.

```bash {filename="charmm.slurm"}
#!/bin/bash
#SBATCH -J charmm
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load md/charmm
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/run.inp .

srun charmm < run.inp > run.out
cp -a run.out "$SLURM_SUBMIT_DIR"/
```

Add `#SBATCH --gpus=…` and `module load md/charmm/48b2+cuda` when using the GPU build.
The CPU module pulls in Intel MPI; keep ranks on **one** node (`srun --cpu-bind=cores`).
