---
title: Trimmomatic
---

[Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) trims adapters and low-quality bases from Illumina FASTQ files (single-end or paired-end).

| | |
| --- | --- |
| Module | `genom/trimmomatic/0.39` |
| Depends on | `lang/java` (loaded automatically) |
| Command | `trimmomatic` (wrapper around the JAR) |
| Adapters | `$TRIMMOMATIC_DIR/adapters` |

## Load

```console
$ module load genom/trimmomatic
```

## Example (paired-end)

Cap Java threads to the Slurm allocation. Without `-threads`, the JVM may try to use every CPU on the node.

```bash {filename="trimmomatic.slurm"}
#!/bin/bash
#SBATCH -J trimmomatic
#SBATCH -c 8
#SBATCH --mem-per-cpu=2G
#SBATCH -t 04:00:00
#SBATCH -o %x-%j.out

module load genom/trimmomatic
export MALLOC_ARENA_MAX=2
trimmomatic PE -threads "$SLURM_CPUS_PER_TASK" \
  "$DATA_DIR/reads/R1.fq.gz" "$DATA_DIR/reads/R2.fq.gz" \
  "$SCRATCH_DIR/trim/R1.paired.fq.gz" "$SCRATCH_DIR/trim/R1.unpaired.fq.gz" \
  "$SCRATCH_DIR/trim/R2.paired.fq.gz" "$SCRATCH_DIR/trim/R2.unpaired.fq.gz" \
  ILLUMINACLIP:$TRIMMOMATIC_DIR/adapters/TruSeq3-PE.fa:2:30:10 \
  LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```

Adapter FASTA files shipped with the install:

- `TruSeq3-PE.fa`, `TruSeq3-PE-2.fa`, `TruSeq3-SE.fa`
- `TruSeq2-PE.fa`, `TruSeq2-SE.fa`
- `NexteraPE-PE.fa`

> [!TIP]
> `MALLOC_ARENA_MAX=2` plus `-threads "$SLURM_CPUS_PER_TASK"` avoids the JVM reserving huge virtual memory on shared nodes.
> See the [Trimmomatic HPC note](https://github.com/usadellab/Trimmomatic#note-for-hpc-users-sge-slurm-lsf-pbs).

Single-end mode is `trimmomatic SE ...`.
Check trimmed reads with [FastQC]({{< relref "fastqc" >}}), then align with [STAR]({{< relref "star" >}}).
