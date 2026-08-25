---
title: Softepigen
---

[Softepigen](https://github.com/franciscoadasme/softepigen/) designs primers for methylation-sensitive high-resolution melting (MS-HRM).
It is developed in the LBQC.

| | |
| --- | --- |
| Module | `genom/softepigen/0.1.0@e9978c2` |
| Command | `softepigen` |

## Load

```console
$ module load genom/softepigen
```

The version string includes a git commit (`@e9978c2`).
Omitting the version loads this default.

## Example

Run on a compute node. Pass your usual input files (sequences / target regions) as documented in the repository:

```bash {filename="softepigen.slurm"}
#!/bin/bash
#SBATCH -J softepigen
#SBATCH -c 4
#SBATCH --mem-per-cpu=4G
#SBATCH -t 02:00:00
#SBATCH -o %x-%j.out

module load genom/softepigen
srun softepigen --help
```

Replace `--help` with the options for your design job.
See the [GitHub README](https://github.com/franciscoadasme/softepigen/) for input formats and flags.

A web front-end may be available on the lab network; the module is the command-line program for batch jobs.
