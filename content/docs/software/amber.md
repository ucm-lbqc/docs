---
title: Amber
---

[Amber](https://ambermd.org/) is a suite for biomolecular simulation (setup with AmberTools, dynamics with `sander` / `pmemd`).

| | |
| --- | --- |
| Module | `md/amber/24` |
| Depends on | `gnu12`, `openmpi4`, `netcdf-fortran` (loaded automatically) |
| Commands | `pmemd`, `pmemd.cuda`, `pmemd.MPI`, `sander`, `cpptraj`, `tleap`, … |
| Home | `$AMBERHOME` |

This install is covered by the lab’s Amber license. Do not copy the binaries off the cluster.

## Load

```console
$ module load md/amber
```

## Example (GPU)

Prefer `pmemd.cuda` on a GPU node ([Job submission]({{< relref "job-submission#gpus" >}})).
Stage the working directory on scratch.

```bash {filename="amber.slurm"}
#!/bin/bash
#SBATCH -J pmemd
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH --gpus=2080ti:1
#SBATCH -t 24:00:00
#SBATCH -o %x-%j.out

module load md/amber
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/{prmtop,inpcrd,mdin} .

srun pmemd.cuda -O -i mdin -o mdout -p prmtop -c inpcrd -r restrt -x mdcrd
cp -a mdout restrt mdcrd "$SLURM_SUBMIT_DIR"/
```

CPU MPI uses `pmemd.MPI` with `#SBATCH -n` and `srun --cpu-bind=cores` (one node only).
Analysis: `cpptraj`.
Setup: `tleap`, `antechamber` (AmberTools).
