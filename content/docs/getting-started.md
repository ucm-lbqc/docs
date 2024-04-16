---
title: Getting Started
next: cluster
weight: 100
---

This section is intended as a brief introduction into HPC and it will guide you through the basics of using the LBQC's cluster.
You'll learn how to connect to the cluster, list and load modules, submit and manage jobs, and accessing user data.
After reading this page, you will hopefully have run your first job successfully in the cluster.
Links are provided throughout the text to point you to more in-depth information on the topic.

{{< callout type="warning" >}}
Basic Linux and HPC knowledge is a prerequisite, which can be learn from many beginner tutorials available online.
{{< /callout >}}

## Getting an account

Users with access to the LBQC's resources are either official members of the LBQC or collaborate with one or more members of the team.

To request an account, you must ask a LBQC member to request an account for you to the administrator.
Once the account request is submitted, the administrator will review the request and then contact you within 1--7 business days.

See the [Accounts]({{< relref "access" >}}) for more details.

## Accessing the cluster

To connect to the cluster, you must log in to the **login node** (named _curie_) from within the university's network.
Open a terminal (Terminal/PowerShell) Use the secure shell (SSH) protocol to connect to the login node as follows:

```console
$ ssh <user>@192.168.57.68
```

where `<user>` is the given user account and `192.168.57.68` is the internal IP address of _curie_.
You must enter your password and then you will be prompted into a login node terminal:

```console
$ ssh <user>@192.168.57.68
fadasme@curie's password:
...

Last login: Wed Apr 10 16:50:48 2024 from 10.212.134.55
[<user>@curie ~]$ hostname
curie
```

Here you can execute commands within _curie_ such as submitting and managing jobs.

{{< callout type="warning" >}}
**Do not run compute-intensive applications from here**.
The login node must be only used for job-related (see below) and simple tasks.
Bad usage will lead to account termination.
{{< /callout >}}

If you want to connect from a remote location (e.g., from your computer at home), you must use a VPN connection to get access to the university's network.
Follow the steps in the [Remote access]({{< ref "access#remote-access" >}}) section to install and configure the VPN in your computer.

See the [Accessing the server]({{< ref access >}}) page for details about how to configure the connection via SSH and some useful tips.

## Managing files

Before running jobs, it is important to understand how to and where to copy your files to the server.
Here is a brief summary about the directories available to you and quick examples how to send files to a remote location.

### Directory structure

The cluster has access to three file systems with different levels of performance, lifetime and available space:

1. The **HOME directory** (`/home/$USER` where `$USER` is your account's username) is used to store your personal files and configurations.
   This directory has limited capacity, so be sure to keep your home directory clean by regularly deleting old data.
   It can be referenced by the `$HOME` environment variable or the `~` shorthand in a path.
2. The **data directory** (`/data/$USER`) is used to store your important and long-term data.
   It is backed up regularly, and it has 1 TB of capacity.
   It can be referenced by the `$DATA_DIR` environment variable.
3. The **scratch directory** (`/scratch/$USER`) is used to store temporary files generated when running a job.
   This folder is stored in a physical disk on the compute nodes (unlike the HOME and data directories) so it is much faster for reading and writing files than the shared file system over the network.
   It can be referenced by the `$SCRATCH_DIR` environment variable.

Here is a summary table of the directory features:

| Directory | Speed | Capacity      | Lifetime   | Backup |
| --------- | ----- | ------------- | ---------- | :----: |
| HOME      | slow  | 100 GB        | 6 months   |   ❌   |
| data      | slow  | 1 TB          | Indefinite |   ✔️   |
| scratch   | fast  | 1 TB (shared) | A week     |   ❌   |

Refer to the [Storage]({{< ref storage >}}) and [Best practices]({{< ref best-practices >}}) pages for more information about how to correctly use the file system.

### Copying files

You can use the SSH protocol to transfer files between your local computer and the cluster.
There are many tools, either command line programs or GUI apps, that work over SSH.
The most widely used are [scp](https://en.wikipedia.org/wiki/Secure_copy_protocol) and [rsync](https://en.wikipedia.org/wiki/Rsync), where the latter is recommended when copying many files.

The following commands show how to copy a file/directory from/to the server.

**Copy a file to the server**

```console
$ scp /path/to/local-file <user>@192.168.57.68:path/to/remote-dir
```

**Copy a file from the server**

```console
$ scp <user>@192.168.57.68:path/to/remote-file /path/to/local-dir
```

**Copy an entire directory to the server**

```console
$ scp -r /path/to/local-dir <user>@192.168.57.68:path/to/remote-dir
```

**Copy an entire directory from the server**

```console
$ scp -r <user>@192.168.57.68:path/to/remote-dir /path/to/local-dir
```

These commands must be entered in a terminal in your local computer running a UNIX-like OS like Linux or macOS.
It is recommended to use the [Windows Subsystem for Linux (WSL)](https://learn.microsoft.com/en-us/windows/wsl/install) under Windows rather than installing third-party applications, if possible.

{{< callout type="warning" >}}
The path following the colon after the remote address is relative to the HOME directory unless is prefixed by `/`, which it is treated as an absolute path instead.
That is, `/path/to/file` points to the file at `/path/to/file`, whereas `path/to/file` points to `$HOME/path/to/file`.
{{< /callout >}}

More information about file transfer can be found on the [File Transfer]({{< ref "storage#file-transfer" >}}) section.

The address in the above commands can be shortened to an alias (e.g., `scp /path/to/local-file lbqc:path/to/remote-dir`) by setting up an SSH configuration file.
Refer to the [SSH customization]({{< ref "access#ssh-customization" >}}) section for details.

## Using software

You can use pre-installed software, build your own software, or submit a software request to get software installed by the admin team.
Here we will focus on using pre-installed software only.
Refer to the [Building software]({{< ref "software#building" >}}) and [Request a software]({{< ref "software#request" >}}) sections for information in these topics.

There are many programs pre-installed in the server.
We use the [Environment Module](https://modules.readthedocs.io/en/latest/) system to manage software packages including different versions and dependencies.
To get a list of all provided packages, run the following command:

```console
$ module avail

--------------------------------------- /opt/ohpc/pub/moduledeps/gnu12-openmpi4 ----------------------------------------
   boost/1.81.0    fftw/3.3.10    phdf5/1.14.0    scalapack/2.2.0

-------------------------------------------- /opt/ohpc/pub/moduledeps/gnu12 --------------------------------------------
   gsl/2.7.1      impi/2021.9.0 (D)    mpich/3.4.3-ofi    openmpi4/4.1.5 (L)
   hdf5/1.14.0    impi/2021.11         openblas/0.3.21

---------------------------------------------- /opt/ohpc/pub/modulefiles -----------------------------------------------
   EasyBuild/4.7.2        intel/2024.0.2             md/gromacs/2024.1+cuda        os
   autotools       (L)    lang/crystal/1.11.2        md/namd/2.14+cuda             pmix/4.2.6
   cmake/3.24.2           lang/python/3.12.2         md/namd/2.14                  prun/2.2               (L)
   cuda/12.3.2            libfabric/1.18.0    (L)    md/namd/3.0b6+cuda     (D)    qm/jdftx/1.7.0@12396ea
   gnu12/12.2.0    (L)    md/charmm/48b2+cuda        mm/schrodinger/2023-4         qm/vasp/6.3.2+zen3
   hwloc/2.9.0     (L)    md/charmm/48b2      (D)    mm/vmd/1.9.4a57               qm/xtb/6.7.0
   intel/2023.1.0  (D)    md/desmond/2023-4          ohpc                   (L)    ucx/1.14.0             (L)

  Where:
   D:  Default Module
   L:  Module is loaded

If the avail list is too long consider trying:

"module --default avail" or "ml -d av" to just list the default modules.
"module overview" or "ml ov" to display the number of modules for each name.

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

Each software package is identified by its name and version separated by a slash, and optionally preceded by its category.
Packages may also specify additional features by appending `+<feature>`.
For instance, `md/namd/2.14+cuda` refers to the NAMD software, version 2.14, compiled with support for GPU acceleration via CUDA, under the Molecular Dynamics (`md`) category.
The combination of the version and enabled features is referred to as a _variant_, that is, `md/namd/2.14+cuda` is a variant of `md/namd/2.14`.

You can pass a name to the above command to only list the matching packages (footer omitted for brevity):

```console
$ module avail namd

---------------------------------------------- /opt/ohpc/pub/modulefiles -----------------------------------------------
   md/namd/2.14+cuda    md/namd/2.14    md/namd/3.0b6+cuda (D)
```

Here we can see that there are three variants of NAMD, two different versions and two variants support GPU acceleration.

Once the desired software variant is identified, let's select `md/namd/2.14+cuda` for this example, you must load the corresponding module to set up the environment and software dependencies:

```console
$ module load md/namd/2.14+cuda
$ which namd2
/opt/ohpc/pub/apps/md/namd/2.14+cuda/namd2
```

Note that the `namd2` executable was not available before loading the module.
This also takes care of the dependencies, such as any library that the software requires to work.

Multiple packages can be loaded all at once:

```console
$ module load md/namd/2.14+cuda lang/python/3.12.2 qm/xtb/6.7.0
```

The categories are set up to ease the lookup of the desired software.
For instance, typing `module load md` and pressing the tab key in the terminal will trigger the autocomplete and show all the matching packages:

```console
$ module load md/
md/charmm               md/desmond              md/gromacs/2024.1+cuda  md/namd/2.14+cuda
md/charmm/48b2          md/desmond/2023-4       md/namd                 md/namd/3.0b6+cuda
md/charmm/48b2+cuda     md/gromacs              md/namd/2.14
```

Finally, the loaded modules can be listed by the following command:

```console
$ module list

Currently Loaded Modules:
  1) md/namd/2.14+cuda    3) tbb/2021.9.0           5) oclfpga/2023.1.0    7) mkl/2023.1.0     9) qm/xtb/6.7.0
  2) lang/python/3.12.2   4) compiler-rt/2023.1.0   6) compiler/2023.1.0   8) intel/2023.1.0
```

Note that there are a lot of extra packages, such as the compiler libraries used during compilation (`intel/2023.1.0`) of one or more packages, which are automatically loaded based on the dependencies specified in the corresponding modules.

{{< callout type="info" >}}
**Module scope**
<br />
The modules you load are only active in your current session.
Therefore, you need to load the desired software each time you open a new session.
{{< /callout >}}

## Your first job

## Data backup

## Other resources

## Submitting interactive jobs

[^1]: Strictly speaking, you should copy the input files to the local scratch to improve performance.
