---
title: samtools
---

[samtools](https://www.htslib.org/) reads, writes, and post-processes SAM/BAM/CRAM alignment files (sort, index, subset, stats).

| | |
| --- | --- |
| Module | `genom/samtools/1.22` |
| Commands | `samtools`, plus small converters in the same `bin` |

## Load

```console
$ module load genom/samtools
```

## Example

```bash {filename="samtools.slurm"}
#!/bin/bash
#SBATCH -J samtools-sort
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 04:00:00
#SBATCH -o %x-%j.out

module load genom/samtools
srun samtools sort -@ "$SLURM_CPUS_PER_TASK" \
  -o "$SCRATCH_DIR/aln.sorted.bam" \
  "$SCRATCH_DIR/aln.sam"
samtools index "$SCRATCH_DIR/aln.sorted.bam"
samtools flagstat "$SCRATCH_DIR/aln.sorted.bam"
```

`-@` is the number of extra compression threads; matching `$SLURM_CPUS_PER_TASK` is a reasonable default.

Index a FASTA for GATK and viewers:

```console
$ samtools faidx "$DATA_DIR/ref/genome.fa"
```

Use [BWA]({{< relref "bwa" >}}) or [STAR]({{< relref "star" >}}) to produce the alignment first.
