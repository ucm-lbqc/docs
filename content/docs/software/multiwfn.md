---
title: Multiwfn
---

[Multiwfn](http://sobereva.com/multiwfn) is a wavefunction-analysis program (population, critical points, surfaces, spectra, and related analyses).

| | |
| --- | --- |
| Module | `qm/multiwfn/2026.7.15` |
| Command | `Multiwfn` |

The module raises the stack size (`ulimit -s unlimited`) and sets `OMP_STACKSIZE`.
Still cap OpenMP threads to your allocation.

## Load

```console
$ module load qm/multiwfn
```

## Example

Interactive Multiwfn is a text menu. For jobs, feed a command file on stdin (see the Multiwfn manual, “silent/batch” mode).

```bash {filename="multiwfn.slurm"}
#!/bin/bash
#SBATCH -J multiwfn
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 04:00:00
#SBATCH -o %x-%j.out

module load qm/multiwfn
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$DATA_DIR"/wfn/molecule.fchk .
cp "$SLURM_SUBMIT_DIR"/multiwfn.inp .

srun Multiwfn molecule.fchk < multiwfn.inp
```

Input wavefunctions often come from [ORCA]({{< relref "orca" >}}) or [Q-Chem]({{< relref "qchem" >}}) (`.molden`, `.fchk`, `.wfn`).
Do not run the interactive menu on the login node for large files.
