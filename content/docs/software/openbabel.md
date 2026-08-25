---
title: Open Babel
---

[Open Babel](https://openbabel.org/) converts and processes chemical file formats (SMILES, PDB, MOL2, XYZ, and many others).

| | |
| --- | --- |
| Module | `mm/openbabel/3.1.1` |
| Commands | `obabel`, plus `obminimize`, `obenergy`, … |
| Data | `$BABEL_DATADIR` |

## Load

```console
$ module load mm/openbabel
```

## Example

```bash {filename="obabel.slurm"}
#!/bin/bash
#SBATCH -J obabel
#SBATCH -c 1
#SBATCH --mem-per-cpu=4G
#SBATCH -t 01:00:00
#SBATCH -o %x-%j.out

module load mm/openbabel
srun obabel "$DATA_DIR/inputs/mols.sdf" -O "$DATA_DIR/results/mols.xyz" -m
```

`-m` splits a multi-molecule file into one output per record.
See `obabel -H` for format codes (`-i` / `-o`).

This is a conversion tool, not a production MD engine.
For simulations see [GROMACS]({{< relref "gromacs" >}}), [NAMD]({{< relref "namd" >}}), or [xTB]({{< relref "xtb" >}}).
