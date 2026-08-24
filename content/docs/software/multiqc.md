---
title: MultiQC
---

[MultiQC](https://seqera.io/multiqc/) scans a directory for logs and reports from other bioinformatics tools and builds a single interactive HTML summary.

| | |
| --- | --- |
| Module | `genom/multiqc/1.35` |
| Command | `multiqc` |

## Load

```console
$ module load genom/multiqc
```

> [!NOTE]
> You do **not** need `lang/conda`.
> The module already points at its own Python.
> FastQC, STAR, or Trimmomatic do not have to be loaded at the same time: MultiQC only reads files that those tools already wrote.

## Example

```bash {filename="multiqc.slurm"}
#!/bin/bash
#SBATCH -J multiqc
#SBATCH -c 2
#SBATCH -t 00:30:00
#SBATCH -o %x-%j.out

module load genom/multiqc
multiqc "$SCRATCH_DIR/qc" --outdir "$DATA_DIR/multiqc"
```

Typical inputs are FastQC HTML/zip files, STAR `Log.final.out`, and Trimmomatic logs in the directory you pass as the first argument.
The default report name is `multiqc_report.html`.

See the [MultiQC documentation](https://docs.seqera.io/multiqc/getting_started/running_multiqc) for extra flags (`--title`, `--filename`, module filters).
