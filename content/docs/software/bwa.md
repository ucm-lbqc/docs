---
title: BWA
---

[BWA](https://bio-bwa.sourceforge.net/) (Burrows–Wheeler Aligner) maps short DNA reads to a reference genome.

| | |
| --- | --- |
| Module | `genom/bwa/0.7.19` |
| Command | `bwa` |

## Load

```console
$ module load genom/bwa
```

## Index (once)

Keep the index next to the FASTA on `$DATA_DIR`. Rebuilding it is expensive.

```bash {filename="bwa-index.slurm"}
#!/bin/bash
#SBATCH -J bwa-index
#SBATCH -c 1
#SBATCH --mem-per-cpu=8G
#SBATCH -t 04:00:00
#SBATCH -o %x-%j.out

module load genom/bwa
srun bwa index "$DATA_DIR/ref/genome.fa"
```

## Align

```bash {filename="bwa-mem.slurm"}
#!/bin/bash
#SBATCH -J bwa-mem
#SBATCH -c 16
#SBATCH --mem-per-cpu=4G
#SBATCH -t 08:00:00
#SBATCH -o %x-%j.out

module load genom/bwa genom/samtools
srun bwa mem -t "$SLURM_CPUS_PER_TASK" \
  "$DATA_DIR/ref/genome.fa" \
  "$DATA_DIR/reads/R1.fq.gz" "$DATA_DIR/reads/R2.fq.gz" \
  | samtools sort -@ "$SLURM_CPUS_PER_TASK" -o "$SCRATCH_DIR/aln.bam"
samtools index "$SCRATCH_DIR/aln.bam"
```

`bwa mem` is the usual algorithm for Illumina reads.
For splicing RNA-seq, use [STAR]({{< relref "star" >}}) instead.
Sort and index the BAM with [samtools]({{< relref "samtools" >}}).
