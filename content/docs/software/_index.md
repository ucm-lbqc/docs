---
title: Software
weight: 600
cascade:
  params:
    reversePagination: false
---

Pre-installed applications on the cluster are managed with [Lmod](https://lmod.readthedocs.io/) environment modules.
Loading a module puts the program on your `PATH` and pulls in any dependencies it needs (compilers, Java, MPI, and so on).

Application guides in this section are listed **alphabetically** in the sidebar.
Each page covers one module: what it is, how to load it, a minimal example, and cluster-specific caveats.

{{< cards >}}
  {{< card link="fastqc" title="FastQC" subtitle="Quality control of FASTQ files" >}}
  {{< card link="multiqc" title="MultiQC" subtitle="Aggregate QC reports into one HTML file" >}}
  {{< card link="sratoolkit" title="SRA Toolkit" subtitle="Download and convert NCBI SRA runs" >}}
  {{< card link="star" title="STAR" subtitle="Splice-aware RNA-seq aligner" >}}
  {{< card link="telocal" title="TElocal" subtitle="Locus-level transposable element quantification" >}}
  {{< card link="trimmomatic" title="Trimmomatic" subtitle="Adapter and quality trimming of Illumina reads" >}}
{{< /cards >}}

> [!WARNING]
> Do not run these tools on the **login node**. Submit a Slurm job (`sbatch` or `salloc`). See [Getting Started]({{< relref "getting-started" >}}).

## Using modules

List everything that is installed:

```console
$ module avail
```

Filter by category or name. Genomics tools live under `genom`:

```console
$ module avail genom
$ module spider STAR
$ module whatis genom/fastqc
$ module help genom/fastqc
```

Load and unload:

```console
$ module load genom/fastqc
$ module list
$ module unload genom/fastqc
$ module purge
```

If you omit the version, Lmod loads the default (marked `(D)` in `module avail`).

> [!NOTE]
> Modules apply only to the current shell.
> Load them again in every new session, and always load them **inside** the job script so the compute node sees the same environment.

You can load several modules in one command. Independent tools (Java binaries, STAR, SRA Toolkit) combine without issue.
MultiQC and TElocal each have their own Python prefix; you do **not** need `lang/conda` to run them.

Categories group related packages (`genom`, `md`, `qm`, `lang`, …). Tab-complete after `module load genom/` to see what is available.

## Storage reminders

| Location | Variable | Use for |
| -------- | -------- | ------- |
| Home | `$HOME` | Config and small files only |
| Data | `$DATA_DIR` | Inputs, genome indices, results you must keep |
| Scratch | `$SCRATCH_DIR` | Temporary job files (deleted after a week) |

Details: [Storage]({{< relref "storage" >}}).

## Example RNA-seq / TE workflow

These tools are often used in **separate jobs**, not all loaded at once.
A typical sequence (adjust to your protocol):

1. [SRA Toolkit]({{< relref "sratoolkit" >}}) — `prefetch` + `fasterq-dump` into scratch, keep FASTQ on `$DATA_DIR` if needed.
2. [FastQC]({{< relref "fastqc" >}}) — QC of raw reads.
3. [Trimmomatic]({{< relref "trimmomatic" >}}) — adapters and quality.
4. [FastQC]({{< relref "fastqc" >}}) again on trimmed reads (optional).
5. [STAR]({{< relref "star" >}}) — build the index once on `$DATA_DIR`; map with multimappers if you will run TElocal.
6. [TElocal]({{< relref "telocal" >}}) — counts from the BAM (download `.locInd` yourself).
7. [MultiQC]({{< relref "multiqc" >}}) — one HTML report from FastQC/STAR logs.

Each step is a `sbatch` script on the corresponding page.
Do not chain a multi-hour STAR run and a download on the login node.

## Building software {#building}

You may compile tools in your own directories (`$HOME` or `$DATA_DIR`) if they are not provided as modules.
Keep large builds and test runs off the login node.

Do not install into `/opt/ohpc/pub`; that tree is reserved for cluster-wide modules.

## Request a software {#request}

If a package should be available to everyone as a module, email the administrator at [fadasme@ucm.cl](mailto:fadasme@ucm.cl) with:

- Name and version (or git tag)
- Homepage or repository
- Why it is needed and who will use it
- Any license constraints
