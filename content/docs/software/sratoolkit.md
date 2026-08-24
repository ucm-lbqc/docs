---
title: SRA Toolkit
---

The [NCBI SRA Toolkit](https://github.com/ncbi/sra-tools) downloads sequencing runs from the Sequence Read Archive and converts them to FASTQ.

| | |
| --- | --- |
| Module | `genom/sratoolkit/3.4.1` |
| Commands | `prefetch`, `fasterq-dump`, `vdb-config` |

Prefer `prefetch` + `fasterq-dump`. `fastq-dump` is legacy and slower.

## Load

```console
$ module load genom/sratoolkit
```

## One-time cache setup

NCBI writes downloaded runs to a user cache.
Point it at **`$DATA_DIR`**, not `$HOME` (quota) and not only `$SCRATCH_DIR` (deleted after a week) if you need to keep the SRA files:

```console
$ module load genom/sratoolkit
$ mkdir -p "$DATA_DIR/ncbi"
$ vdb-config --set "/repository/user/main/public/root=$DATA_DIR/ncbi"
```

> [!TIP]
> Use `vdb-config --set` on the cluster.
> `vdb-config -i` is an interactive TUI and is awkward over SSH.

## Example

SRA runs are often tens to hundreds of gigabytes. Run this in Slurm, not on the login node:

```bash {filename="sra.slurm"}
#!/bin/bash
#SBATCH -J sra-download
#SBATCH -c 8
#SBATCH -t 04:00:00
#SBATCH -o %x-%j.out

module load genom/sratoolkit
ACC=SRR390728
prefetch "$ACC" --output-directory "$SCRATCH_DIR/sra"
fasterq-dump "$SCRATCH_DIR/sra/$ACC" \
  --outdir "$SCRATCH_DIR/fastq" \
  --threads "$SLURM_CPUS_PER_TASK"
```

Copy keepers from scratch to `$DATA_DIR` before the week is up.
Then run [FastQC]({{< relref "fastqc" >}}) on the FASTQ files.

Official install notes: [NCBI SRA Toolkit wiki](https://github.com/ncbi/sra-tools/wiki/02.-Installing-SRA-Toolkit) (binaries are already provided here; you only need to load the module).
