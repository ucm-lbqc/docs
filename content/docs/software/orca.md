---
title: ORCA
---

[ORCA](https://www.faccts.de/orca) is a quantum-chemistry package (SCF, DFT, correlated methods, spectroscopy).

| | |
| --- | --- |
| Modules | `qm/orca/6.1.1` (default among 6.x), also `6.1.1+avx2`, `6.0.1`, `6.0.0`, `5.0.4` |
| Depends on | `gnu12`, `openmpi4` |
| Command | `orca` |
| Home | `$ORCA_DIR` |

Academic license (EULA in the install tree). Use it only under that license; do not copy the binaries.

```console
$ module avail qm/orca
```

`+avx2` is a CPU-tuned build; the plain `6.1.1` is the safe default if you are unsure.

## Load

```console
$ module load qm/orca
```

## Parallel runs

Set the number of processes **in the input** (`%pal nprocs …`) to match `#SBATCH -n`.
Let ORCA start MPI itself. Do **not** wrap the command in `srun` or `mpirun` (nested MPI).

```text
# job.inp (excerpt)
! B3LYP def2-SVP
%pal nprocs 16 end
* xyzfile 0 1 geom.xyz
```

```bash {filename="orca.slurm"}
#!/bin/bash
#SBATCH -J orca
#SBATCH -n 16
#SBATCH --mem-per-cpu=4G
#SBATCH -t 24:00:00
#SBATCH -o %x-%j.out

module load qm/orca
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/job.inp "$SLURM_SUBMIT_DIR"/geom.xyz .

$ORCA_DIR/orca job.inp > job.out
cp -a job.out job.gbw "$SLURM_SUBMIT_DIR"/
```

Use the full path `$ORCA_DIR/orca` so the OpenMPI wrapper named `orca` cannot shadow the program.
Keep `-n` on **one** node.
Scratch files stay in `$WORKDIR`; copy `.gbw` / `.hess` keepers to `$DATA_DIR`.

Wavefunction analysis: [Multiwfn]({{< relref "multiwfn" >}}).
