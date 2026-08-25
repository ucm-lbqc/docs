---
title: NAMD
---

[NAMD](https://www.ks.uiuc.edu/Research/namd/) is a parallel molecular-dynamics code for large biomolecular systems.

| | |
| --- | --- |
| Modules | `md/namd/…` (several versions; `+cuda` and `+avx512` variants) |
| Commands | `namd2` (2.14), `namd3` (3.x) |

List what is installed:

```console
$ module avail md/namd
```

Omit the version to load the default (marked `(D)`).
For production on a GPU node, pick a **`+cuda`** variant explicitly, for example:

```console
$ module load md/namd/3.0.1+cuda
```

`+avx512` builds need a CPU that supports AVX-512 (the large Intel/AMD nodes; not every box).
If you are unsure, use a `+cuda` or plain build.

NAMD is free for academic use; do not redistribute the binaries.

## Example (GPU, NAMD 3)

```bash {filename="namd.slurm"}
#!/bin/bash
#SBATCH -J namd
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH --gpus=2080ti:1
#SBATCH -t 24:00:00
#SBATCH -o %x-%j.out

module load md/namd/3.0.1+cuda
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/*.namd "$SLURM_SUBMIT_DIR"/*.pdb "$SLURM_SUBMIT_DIR"/*.psf . 2>/dev/null || true

srun namd3 +p"$SLURM_CPUS_PER_TASK" +setcpuaffinity eq.namd
```

NAMD 2.14 uses `namd2` instead of `namd3`.
`+p` is the number of threads; keep it equal to `-c`.
One node only.

Setup and visualization often use [VMD]({{< relref "vmd" >}}).
