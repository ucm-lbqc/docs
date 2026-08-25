---
title: STAR
---

[STAR](https://github.com/alexdobin/STAR) (Spliced Transcripts Alignment to a Reference) is a splice-aware RNA-seq aligner.

| | |
| --- | --- |
| Module | `genom/star/2.7.11b` |
| Commands | `STAR`, `STARlong` |

## Load

```console
$ module load genom/star
```

## Memory and indices

Mammalian genomes need about **32 GB of RAM** for mapping; building the index can need more.
Request it with `--mem-per-cpu` (the cluster default is ~4 GB per CPU).

Store the genome index in **`$DATA_DIR`**. Scratch is wiped after a week and regenerating an index is expensive.

## Generate an index

```bash {filename="star-index.slurm"}
#!/bin/bash
#SBATCH -J star-index
#SBATCH -c 16
#SBATCH --mem-per-cpu=4G
#SBATCH -t 06:00:00
#SBATCH -o %x-%j.out

module load genom/star
STAR --runThreadN "$SLURM_CPUS_PER_TASK" \
  --runMode genomeGenerate \
  --genomeDir "$DATA_DIR/star_index" \
  --genomeFastaFiles "$DATA_DIR/ref/genome.fa" \
  --sjdbGTFfile "$DATA_DIR/ref/genes.gtf"
```

## Align reads

```bash {filename="star-align.slurm"}
#!/bin/bash
#SBATCH -J star-align
#SBATCH -c 16
#SBATCH --mem-per-cpu=4G
#SBATCH -t 08:00:00
#SBATCH -o %x-%j.out

module load genom/star
mkdir -p "$SCRATCH_DIR/star"
STAR --runThreadN "$SLURM_CPUS_PER_TASK" \
  --genomeDir "$DATA_DIR/star_index" \
  --readFilesIn "$DATA_DIR/reads/R1.fq.gz" "$DATA_DIR/reads/R2.fq.gz" \
  --readFilesCommand zcat \
  --outSAMtype BAM SortedByCoordinate \
  --outFileNamePrefix "$SCRATCH_DIR/star/"
```

### For TElocal

TElocal needs multimappers. Add the following (Hammell lab recommendation) when the BAM will be used with [TElocal]({{< relref "telocal" >}}):

```bash
--outFilterMultimapNmax 100 \
--winAnchorMultimapNmax 100
```

STAR's `Log.final.out` can be picked up by [MultiQC]({{< relref "multiqc" >}}).
