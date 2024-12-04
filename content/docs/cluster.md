---
title: LBQC cluster
next: accounts
weight: 200
---

## Login & compute nodes

## Hardware

| Name  | CPU                                                          | RAM       | Storage              | GPU                                                       |
| ----- | ------------------------------------------------------------ | --------- | -------------------- | --------------------------------------------------------- |
| nc01  | 2x Intel® Xeon® Silver 4216 16-core @ 2.1GHz 22 MB Cache     | 256GB     | SSD 1TB              |                                                           |
| nc02  | 2x Intel® Xeon® Silver 4216 16-core @ 2.1GHz 22 MB Cache     | 256GB     | SSD 1TB              |                                                           |
| nc03  | 2x Intel® Xeon® Silver 4216 16-core @ 2.1GHz 22 MB Cache     | 256GB     | SSD 1TB              |                                                           |
| nc04  | 2x Intel® Xeon® Silver 4216 16-core @ 2.1GHz 22 MB Cache     | 256GB     | SSD 1TB              |                                                           |
| wc01  | Intel® Xeon® Silver 4216 16-core @ 2.10GHz 22 MB Cache       | 128GB     | SSD 1TB              | NVIDIA GeForce RTX 2080 Ti, NVIDIA GeForce RTX 2080 Super |
| maria | 2x AMD EPYC 7713 64-Core @ 2.0GHz 256 MB Cache               | 320GB     | NVMe 1TB             |                                                           |
| rose  | 2x AMD EPYC 7713 64-Core @ 2.0GHz 256 MB Cache               | 480GB     | NVMe 1TB             |                                                           |
| sina  | 2x Intel® Xeon® Silver 4416+ 20-core @ 2.0 GHz 37.5 MB Cache | 512GB     | NVMe 1TB             | 2x NVIDIA L4                                              |
| **8** | **440 cores**                                                | **2.7TB** | **48TB**<sup>†</sup> | **4 GPUs**                                                |

<sup>†</sup> Hosted in a high-performance NAS QNAP TS-673A 8GB with RAID 6 distributed via NFS.
See the [Storage]({{< ref storage >}}) page for details.
