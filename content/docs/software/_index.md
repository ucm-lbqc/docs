---
title: Software
weight: 600
cascade:
  params:
    reversePagination: false
---

Pre-installed **scientific applications** on the cluster are managed with [Lmod](https://lmod.readthedocs.io/) environment modules.
Loading a module puts the program on your `PATH` and pulls in any dependencies it needs (compilers, Java, MPI, CUDA, and so on).

This section documents production science codes only (genomics, molecular dynamics, modeling, quantum chemistry).
Compilers, MPI, CUDA, CMake, Conda, Java, and Python are loaded as **dependencies**; they are not given their own guides.
List everything with `module avail`.

Application pages are **alphabetical** in the sidebar.
Each page covers one module (or a family of versions): what it is, how to load it, a minimal job, and cluster-specific caveats.

> [!WARNING]
> Do not run these tools on the **login node**. Submit a Slurm job (`sbatch` or `salloc`). See [Getting Started]({{< relref "getting-started" >}}).

## Genomics

{{< cards >}}
  {{< card link="bwa" title="BWA" subtitle="Short-read DNA alignment" >}}
  {{< card link="fastqc" title="FastQC" subtitle="Quality control of FASTQ files" >}}
  {{< card link="gatk" title="GATK" subtitle="Variant discovery toolkit" >}}
  {{< card link="multiqc" title="MultiQC" subtitle="Aggregate QC reports into one HTML file" >}}
  {{< card link="samtools" title="samtools" subtitle="SAM/BAM/CRAM processing" >}}
  {{< card link="snpeff" title="SnpEff" subtitle="Variant effect annotation" >}}
  {{< card link="softepigen" title="Softepigen" subtitle="MS-HRM primer design" >}}
  {{< card link="sratoolkit" title="SRA Toolkit" subtitle="Download and convert NCBI SRA runs" >}}
  {{< card link="star" title="STAR" subtitle="Splice-aware RNA-seq aligner" >}}
  {{< card link="telocal" title="TElocal" subtitle="Locus-level transposable element quantification" >}}
  {{< card link="trimmomatic" title="Trimmomatic" subtitle="Adapter and quality trimming of Illumina reads" >}}
{{< /cards >}}

## Molecular dynamics

{{< cards >}}
  {{< card link="amber" title="Amber" subtitle="Biomolecular simulation suite" >}}
  {{< card link="charmm" title="CHARMM" subtitle="Macromolecular mechanics" >}}
  {{< card link="desmond" title="Desmond" subtitle="GPU molecular dynamics (D. E. Shaw)" >}}
  {{< card link="gromacs" title="GROMACS" subtitle="Molecular dynamics with CUDA" >}}
  {{< card link="lammps" title="LAMMPS" subtitle="Classical MD for materials" >}}
  {{< card link="namd" title="NAMD" subtitle="Parallel biomolecular MD" >}}
  {{< card link="openmm" title="OpenMM" subtitle="MD toolkit (Python / CUDA)" >}}
{{< /cards >}}

## Molecular modeling

{{< cards >}}
  {{< card link="openbabel" title="Open Babel" subtitle="Chemical file format conversion" >}}
  {{< card link="plumed" title="PLUMED" subtitle="Enhanced sampling and MD analysis" >}}
  {{< card link="schrodinger" title="Schrödinger" subtitle="Commercial modeling suite" >}}
  {{< card link="vmd" title="VMD" subtitle="Visualization and analysis" >}}
{{< /cards >}}

## Quantum chemistry

{{< cards >}}
  {{< card link="dftbplus" title="DFTB+" subtitle="Density-functional tight binding" >}}
  {{< card link="jdftx" title="JDFTx" subtitle="Plane-wave DFT" >}}
  {{< card link="multiwfn" title="Multiwfn" subtitle="Wavefunction analysis" >}}
  {{< card link="orca" title="ORCA" subtitle="Quantum chemistry package" >}}
  {{< card link="qchem" title="Q-Chem" subtitle="Ab initio quantum chemistry" >}}
  {{< card link="vasp" title="VASP" subtitle="Plane-wave DFT (Zen 3 build)" >}}
  {{< card link="xtb" title="xTB" subtitle="Semiempirical tight binding" >}}
{{< /cards >}}

## Using modules

List everything that is installed:

```console
$ module avail
```

Filter by category or name:

```console
$ module avail genom
$ module avail md
$ module avail qm
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
Several packages have **variants** in the version string (`+cuda`, `+avx2`, `+zen3`). Load the variant you need; the default is not always the GPU build.

> [!NOTE]
> Modules apply only to the current shell.
> Load them again in every new session, and always load them **inside** the job script so the compute node sees the same environment.

You can load several modules in one command when the tools do not share a Python/conda prefix.
MultiQC, TElocal, and OpenMM each have their own prefix; you do **not** need `lang/conda` to run them, and you should not mix those prefixes in the same job.

Categories group related packages (`genom`, `md`, `mm`, `qm`, `lang`, …). Tab-complete after `module load md/` to see what is available.

## Storage reminders

| Location | Variable | Use for |
| -------- | -------- | ------- |
| Home | `$HOME` | Config and small files only |
| Data | `$DATA_DIR` | Inputs, indices, results you must keep |
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

For **DNA** short reads, a common chain is [BWA]({{< relref "bwa" >}}) → [samtools]({{< relref "samtools" >}}) → [GATK]({{< relref "gatk" >}}) → [SnpEff]({{< relref "snpeff" >}}).

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
