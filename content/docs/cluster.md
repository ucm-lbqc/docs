---
title: LBQC cluster
next: accounts
weight: 200
---

The LBQC has a high-performance computing (HPC) server formed by a cluster of several computing nodes.

The nodes have been acquired with funding from:

- Faculty of Medicine, Universidad Católica del Maule, Chile.
- ANID (SIA SA772100091), Chile.

## Hardware

| Name  | CPU                                                          | RAM       | GPU                                                       |
| ----- | ------------------------------------------------------------ | --------- | --------------------------------------------------------- |
| login | Intel® Xeon® E-2276G _6C_ 3.80GHz 12 MB Cache                | 64 GB     |                                                           |
| nc01  | 2x Intel® Xeon® Silver 4216 16-core @ 2.1GHz 22 MB Cache     | 256GB     |                                                           |
| nc02  | 2x Intel® Xeon® Silver 4216 16-core @ 2.1GHz 22 MB Cache     | 256GB     |                                                           |
| nc03  | 2x Intel® Xeon® Silver 4216 16-core @ 2.1GHz 22 MB Cache     | 256GB     |                                                           |
| nc04  | 2x Intel® Xeon® Silver 4216 16-core @ 2.1GHz 22 MB Cache     | 256GB     |                                                           |
| wc01  | Intel® Xeon® Silver 4216 16-core @ 2.10GHz 22 MB Cache       | 128GB     | NVIDIA GeForce RTX 2080 Ti, NVIDIA GeForce RTX 2080 Super |
| maria | 2x AMD EPYC 7713 64-Core @ 2.0GHz 256 MB Cache               | 320GB     |                                                           |
| rose  | 2x AMD EPYC 7713 64-Core @ 2.0GHz 256 MB Cache               | 480GB     |                                                           |
| sina  | 2x Intel® Xeon® Silver 4416+ 20-core @ 2.0 GHz 37.5 MB Cache | 512GB     | 2x NVIDIA L4                                              |
| **8** | **440 cores**                                                | **2.7TB** | **4 GPUs**                                                |

Backed up by 2x UPS APC Smart-UPS On-Line, 10kVA/10kW.

The storage is divided in three levels:

- **Distributed data**:
  - Login node OS (NVMe 512GB).
  - User directories `/home/<user>` (2x HDD 6.0TB in RAID 1) mounted via NFS.
  - Software (2x HDD 2.0TB in RAID 1) mounted via NFS.
- **Scratch** `/scratch/<user>` directory (SSD 1TB) on compute nodes only. #text(red)[*Files last for a week*].
- **Data** `/data/<user>` directory hosted in a high-performance NAS QNAP TS-673A 8GB _48TB_ with RAID 6 mounted via NFS.

See the [Storage]({{< ref storage >}}) page for details.

## Software stack

The following software is used for server management:

- Base OS: Rocky Linux 9.3.
- Provisioning system: Warewulf 4.5.0rc2.
- Resource management: SLURM 22.05.11.
- Package management: OpenHPC 3.0.0.
- Software management: Lmod 8.7.14.
