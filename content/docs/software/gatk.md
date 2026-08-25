---
title: GATK
---

The [Genome Analysis Toolkit](https://gatk.broadinstitute.org/) (GATK) is a Java suite for variant discovery and related genomics workflows.

| | |
| --- | --- |
| Module | `genom/gatk/4.6.2.0` |
| Depends on | `lang/java` (loaded automatically) |
| Command | `gatk` |

## Load

```console
$ module load genom/gatk
```

## Example

Cap the Java heap to the memory you reserved.
A common first tool is `HaplotypeCaller` on a coordinate-sorted BAM (see [BWA]({{< relref "bwa" >}}) and [samtools]({{< relref "samtools" >}})).

```bash {filename="gatk-hc.slurm"}
#!/bin/bash
#SBATCH -J gatk-hc
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load genom/gatk
export MALLOC_ARENA_MAX=2
srun gatk --java-options "-Xmx24g" HaplotypeCaller \
  -R "$DATA_DIR/ref/genome.fa" \
  -I "$SCRATCH_DIR/aln.bam" \
  -O "$DATA_DIR/results/sample.vcf.gz"
```

`-Xmx` should stay below the total RAM of the job (here 8 × 4 GB).
Leave a few gigabytes for the OS and native libraries.

Reference FASTA files need a `.fai` index (`samtools faidx`) and usually a GATK dictionary (`gatk CreateSequenceDictionary`).

Official tool docs: [GATK Tool Index](https://gatk.broadinstitute.org/hc/en-us/articles/360035890771).
