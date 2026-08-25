---
title: Q-Chem
---

[Q-Chem](https://www.q-chem.com/) is an ab initio quantum-chemistry package.

| | |
| --- | --- |
| Module | `qm/qchem/6.1.1` |
| Command | `qchem` |
| Home | `$QC` / `$QCHEM_DIR` |
| Scratch | `$QCSCRATCH` (set to `$SCRATCH_DIR`) |

Licensed via FlexNet on the login node. Use it only under the lab’s Q-Chem license; do not copy the binaries.

The module sets `QCMPI=seq` (single process unless you change it).
Scratch is already `$SCRATCH_DIR`.

## Load

```console
$ module load qm/qchem
```

## Example

```bash {filename="qchem.slurm"}
#!/bin/bash
#SBATCH -J qchem
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 24:00:00
#SBATCH -o %x-%j.out

module load qm/qchem
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
# Optional SMP parallel (see Q-Chem manual for $QCTHREADS / QCMPI):
export QCTHREADS=$SLURM_CPUS_PER_TASK

qchem job.in job.out
cp -a job.out "$SLURM_SUBMIT_DIR"/
```

Put `job.in` in the submit directory (or copy it to scratch first).
GPU features (BrianQC) need `#SBATCH --gpus=…` and a node that has an NVIDIA GPU.

If the job cannot check out a license, wait and retry, or contact [fadasme@ucm.cl](mailto:fadasme@ucm.cl).
