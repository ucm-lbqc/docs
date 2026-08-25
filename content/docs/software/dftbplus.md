---
title: DFTB+
---

[DFTB+](https://www.dftbplus.org/) runs fast quantum-mechanical atomistic calculations with the density-functional tight-binding method.

| | |
| --- | --- |
| Module | `qm/dftb+/24.1` |
| Depends on | `gnu12`, OpenBLAS, [PLUMED]({{< relref "plumed" >}}) `2.10a` |
| Command | `dftb+` |
| SK files | `$DFTBPLUS_PARAM_DIR` (3ob-3-1 by default) |

## Load

```console
$ module load qm/dftb+
```

The `+` is part of the module name; quote it in some shells if tab-completion misbehaves: `module load 'qm/dftb+/24.1'`.

## Example

DFTB+ reads `dftb_in.hsd` in the current directory.

```bash {filename="dftb.slurm"}
#!/bin/bash
#SBATCH -J dftb
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load qm/dftb+
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/dftb_in.hsd .
cp "$SLURM_SUBMIT_DIR"/*.gen "$SLURM_SUBMIT_DIR"/*.xyz . 2>/dev/null || true

srun dftb+
cp -a detailed.out "$SLURM_SUBMIT_DIR"/
```

Point Slater–Koster files at `$DFTBPLUS_PARAM_DIR` or copy a parameter set you are allowed to use into the work directory.
See the [DFTB+ manual](https://dftbplus.org/documentation).
