---
title: FastQC
---

[FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) checks raw high-throughput sequence data (FASTQ) before and after trimming.

| | |
| --- | --- |
| Module | `genom/fastqc/0.12.1` |
| Depends on | `lang/java` (loaded automatically) |
| Command | `fastqc` |

## Load

```console
$ module load genom/fastqc
```

## Example

Always pass input files so FastQC runs in non-interactive (batch) mode:

```bash {filename="fastqc.slurm"}
#!/bin/bash
#SBATCH -J fastqc
#SBATCH -c 4
#SBATCH -t 01:00:00
#SBATCH -o %x-%j.out

module load genom/fastqc
mkdir -p "$SCRATCH_DIR/qc"
fastqc --threads "$SLURM_CPUS_PER_TASK" \
  --outdir "$SCRATCH_DIR/qc" \
  "$DATA_DIR/reads/"*.fastq.gz
```

> [!WARNING]
> Running `fastqc` with no arguments opens the GUI, which fails on compute nodes.
> Always list the FASTQ files (or directories) on the command line.

By default FastQC writes an HTML report and a zip of plots per file.
Add `--extract` if a later tool needs the unzipped folder.
Use `--outdir` so reports are not scattered next to the reads.

After several samples have been processed, summarise them with [MultiQC]({{< relref "multiqc" >}}).
