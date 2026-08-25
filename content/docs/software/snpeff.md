---
title: SnpEff
---

[SnpEff](https://pcingola.github.io/SnpEff/) annotates variants in a VCF with predicted functional effects.
[SnpSift](https://pcingola.github.io/SnpEff/snpsift/introduction/) (same module) filters and manipulates those VCFs.

| | |
| --- | --- |
| Module | `genom/snpeff/5.2f` |
| Depends on | `lang/java` (loaded automatically) |
| Commands | `snpEff`, `snpSift` |

## Load

```console
$ module load genom/snpeff
```

## Databases

SnpEff needs a pre-built database for the genome you used in variant calling.
List what is already cached, or download one (large; do this in a job, not on the login node):

```console
$ snpEff databases | less
$ snpEff download GRCh38.99
```

Put downloads on `$DATA_DIR` if you need to keep them.

## Example

```bash {filename="snpeff.slurm"}
#!/bin/bash
#SBATCH -J snpeff
#SBATCH -c 4
#SBATCH --mem-per-cpu=4G
#SBATCH -t 04:00:00
#SBATCH -o %x-%j.out

module load genom/snpeff
export MALLOC_ARENA_MAX=2
srun snpEff -Xmx12g GRCh38.99 \
  "$DATA_DIR/results/sample.vcf.gz" \
  > "$DATA_DIR/results/sample.ann.vcf"
```

Replace `GRCh38.99` with the database that matches your reference.
Cap `-Xmx` below the job’s total memory.
Variant calling is typically [GATK]({{< relref "gatk" >}}).
