---
title: OpenMM
---

[OpenMM](https://openmm.org/) is a toolkit for molecular simulation with CPU and CUDA backends.
It is used as a Python library and from the `openmm` scripts in the prefix.

| | |
| --- | --- |
| Module | `md/openmm/8.2.0` |
| Command | `python` from this prefix (`import openmm`) |
| Home | `$OPENMM_HOME` |

You do **not** need `lang/conda` or `lang/python`.
The module’s `PATH` already points at this install’s Python.
Do not mix it with other conda-based modules in the same job.

Pinned to **CUDA 12.4** (driver 550 on the GPU nodes).

## Load

```console
$ module load md/openmm
```

Check devices (run this **inside** a GPU allocation):

```console
$ python -m openmm.testInstallation
```

## Example

```bash {filename="openmm.slurm"}
#!/bin/bash
#SBATCH -J openmm
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH --gpus=l4:1
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load md/openmm
WORKDIR="$SCRATCH_DIR/$SLURM_JOB_ID"
mkdir -p "$WORKDIR"
cd "$WORKDIR"
cp "$SLURM_SUBMIT_DIR"/run.py .

srun python run.py
```

In `run.py`, select the CUDA platform (`Platform.getPlatformByName("CUDA")`) when you requested a GPU.
Copy trajectories back to `$DATA_DIR` when the job finishes.
