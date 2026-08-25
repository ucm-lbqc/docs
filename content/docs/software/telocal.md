---
title: TElocal
---

[TElocal](https://github.com/mhammell-laboratory/TElocal) assigns RNA-seq reads to genes and transposable elements at **locus** resolution.

| | |
| --- | --- |
| Module | `genom/telocal/1.1.3` |
| Command | `TElocal` |
| Upstream | [mghlab.org/software/telocal](https://www.mghlab.org/software/telocal) |

## Load

```console
$ module load genom/telocal
```

> [!NOTE]
> You do **not** need `lang/conda`.
> Align with [STAR]({{< relref "star" >}}) (multimappers enabled) before running TElocal.

## Annotation files are not installed

TElocal needs a gene GTF and a TE **`.locInd`** index.
Those files are large and are **not** bundled with the module.
Download curated indices from the [Hammell lab page](https://www.mghlab.org/software/telocal) into `$DATA_DIR`.

Human samples typically need **20–30 GB of RAM**.

## Example

Use `--sortByPos` when the BAM is sorted by coordinate (STAR default with `--outSAMtype BAM SortedByCoordinate`).
Otherwise the BAM must be unsorted or sorted by query name.

```bash {filename="telocal.slurm"}
#!/bin/bash
#SBATCH -J telocal
#SBATCH -c 8
#SBATCH --mem-per-cpu=4G
#SBATCH -t 12:00:00
#SBATCH -o %x-%j.out

module load genom/telocal
TElocal --sortByPos \
  -b "$SCRATCH_DIR/star/Aligned.sortedByCoord.out.bam" \
  --GTF "$DATA_DIR/ref/genes.gtf" \
  --TE "$DATA_DIR/ref/te_annots.locInd" \
  --project sample_sorted_test
```

Optional: `--stranded no|forward|reverse` (default `no`), `--mode uniq|multi` (default `multi`).
See `TElocal --help` and the [PyPI page](https://pypi.org/project/TElocal/).
