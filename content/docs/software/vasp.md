---
title: VASP
---

[VASP](https://www.vasp.at/) (Vienna Ab initio Simulation Package) is a plane-wave DFT code.

| | |
| --- | --- |
| Module | `qm/vasp/6.3.2+zen3` |
| Depends on | Intel oneAPI, `openmpi4`, ScaLAPACK |
| Commands | `vasp_std`, `vasp_gam`, `vasp_ncl` (plus GNU-suffixed builds in the same `bin`) |
| Home | `$VASP_DIR` |

Licensed software. Use it only under the lab’s VASP license; do not copy POTCARs or binaries off the cluster.

This build is **optimized for AMD Zen 3** (EPYC 7003: `rose`, `maria`).
Ask for that CPU family so you do not land on an Intel node:

```bash
#SBATCH --constraint=cpu-zen3
```

## Load

```console
$ module load qm/vasp
```

## Example

Bind MPI ranks to cores. Hyper-threading is off; without binding, VASP is often very slow.

```bash {filename="vasp.slurm"}
#!/bin/bash
#SBATCH -J vasp
#SBATCH -n 32
#SBATCH --mem-per-cpu=4G
#SBATCH --constraint=cpu-zen3
#SBATCH -t 24:00:00
#SBATCH -o %x-%j.out

module load qm/vasp
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/{INCAR,KPOINTS,POSCAR,POTCAR} .

srun --cpu-bind=cores vasp_std
cp -a OUTCAR CONTCAR vasprun.xml OSZICAR "$SLURM_SUBMIT_DIR"/
```

- `vasp_std` — usual 3D periodic cells  
- `vasp_gam` — Γ-only  
- `vasp_ncl` — non-collinear / spin–orbit  

Keep `-n` on **one** node (max 128 on `rose`/`maria`).
POTCAR files stay in the job directory; they are licensed, not public data.
