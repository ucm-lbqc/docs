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

In a nutshell, the cluster has a directory structure consisting of three levels:

1. The **HOME directory** (`/home/$USER` where `$USER` is your account's username) is used to store your personal files and configurations.
   This directory has limited capacity, so be sure to keep your home directory clean by regularly deleting old data.
   It can be referenced by the `$HOME` environment variable or the `~` shorthand in a path.
2. The **data directory** (`/data/$USER`) is used to store your important and long-term data.
   It is backed up regularly and it has 1 TB of capacity.
   It can be referenced by the `$DATA_DIR` environment variable.
3. The **scratch directory** (`/scratch/$USER`) is used to store temporary files generated when running a job.
   This folder is stored in a physical disk on the compute nodes (unlike the HOME and data directories) so it is much faster for reading and writing files than the shared file system over the network.
   It can be referenced by the `$SCRATCH_DIR` environment variable.

| Name | Speed | Capacity         | Backup |
| ---- | ----- | ---------------- | ------ |
| HOME | slow  | small (a few GB) | ❌     |
| HOME | slow  | small (a few GB) | ❌     |
| HOME | slow  | small (a few GB) | ❌     |

Refer to the [Storage]({{< ref storage >}}) and [Best practices]({{< ref best-practices >}}) pages for more information about how to correctly use the file system.

### Copying files

## Using software

## Your first job

## Data backup

## Other resources

## Submitting interactive jobs

[^1]: Strictly speaking, you should copy the input files to the local scratch to improve performance.
