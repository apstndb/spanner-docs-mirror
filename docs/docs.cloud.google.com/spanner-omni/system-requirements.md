---
name: documents/docs.cloud.google.com/spanner-omni/system-requirements
uri: https://docs.cloud.google.com/spanner-omni/system-requirements
title: Spanner Omni system requirements
description: System requirements for deploying Spanner Omni on-premises and in cloud environments.
data_source: docs.cloud.google.com
update_time: "2026-05-08T21:33:01Z"
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document describes the hardware and software requirements for deploying Spanner Omni on-premises and in cloud environments.

## On-premises requirements

Deploying Spanner Omni on-premises requires the following:

### Hardware requirements

The right hardware helps Spanner Omni and the CLI run reliably and efficiently on-premises. Check these minimum and recommended configurations to optimize your deployment.

#### Server hardware

Server hardware should meet the following recommended configurations:

<table>
<colgroup>
<col style="width: 30%" />
<col style="width: 70%" />
</colgroup>
<thead>
<tr class="header">
<th>OS and platform</th>
<th>Recommended hardware configuration</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Linux</strong></td>
<td><ul>
<li>x86-64 CPU</li>
<li>4 GB of RAM for every vCPU allocated to Spanner</li>
<li>20 or more GB of disk space</li>
</ul></td>
</tr>
<tr class="even">
<td><strong>macOS</strong> (Developer version)</td>
<td><ul>
<li>M1, M2, or M3 CPU</li>
<li>4 GB of RAM</li>
<li>10 GB of disk space</li>
</ul></td>
</tr>
</tbody>
</table>

#### Spanner Omni CLI hardware

The Spanner Omni CLI should meet the following recommended hardware configurations:

<table>
<colgroup>
<col style="width: 30%" />
<col style="width: 70%" />
</colgroup>
<thead>
<tr class="header">
<th>OS and platform</th>
<th>Recommended hardware configuration</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Linux</strong></td>
<td><ul>
<li>x86-64 or ARM CPU</li>
<li>256 MB of RAM</li>
<li>2 GB of disk space</li>
</ul></td>
</tr>
<tr class="even">
<td><strong>macOS</strong></td>
<td><ul>
<li>M1, M2, or M3 CPU</li>
<li>256 MB of RAM</li>
<li>2 GB of disk space</li>
</ul></td>
</tr>
</tbody>
</table>

### Software requirements

For stable and secure operation, Spanner Omni needs supported operating systems and container environments. Confirm your environment meets these software specifications before you proceed.

#### Server software

Server software should meet the following requirements:

<table>
<colgroup>
<col style="width: 30%" />
<col style="width: 70%" />
</colgroup>
<thead>
<tr class="header">
<th>OS and platform</th>
<th>Recommended software requirements</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Linux</strong></td>
<td><ul>
<li>RHEL 9, Ubuntu 22</li>
<li>Linux kernel version 5.3+</li>
<li>For container deployments:
<ul>
<li>Docker Engine 24.0+</li>
<li>Podman 3.0+</li>
<li>Kubernetes 1.31+</li>
</ul></li>
</ul></td>
</tr>
<tr class="even">
<td><strong>macOS</strong> (Developer version)</td>
<td><ul>
<li>macOS 14.7+</li>
<li>Docker Engine 24.0+</li>
<li>Podman 3.0+</li>
</ul></td>
</tr>
</tbody>
</table>

#### Spanner Omni CLI software

The Spanner Omni CLI software should meet the following requirements:

| OS and platform | Software requirements |
| --------------- | --------------------- |
| **Linux**       | RHEL 8+, Ubuntu 20+   |
| **macOS**       | macOS 14.7+           |

### Storage

Proper storage setup is important for data durability and high performance. Use these recommendations for SSDs, file systems, and disk I/O to meet high-availability standards.

  - For each VM, use a dedicated, persistent, and attachable solid-state drive (SSD) with an `ext4` file system to store your data.
  - Allocate 500 GB of storage per vCPU.
  - Make sure the storage is durable enough for a high-availability system.
  - Spanner Omni is tested with Dell PowerFlex block storage, which is recommended for production usage.
  - Local disks aren't supported.

#### Disk I/O

Your disk I/O should achieve 500 IOPS and 30 MB per second per vCPU.

## Cloud requirements

Deploying Spanner Omni in cloud environments requires the following:

### Google Cloud

Review the compute and storage specifications for Google Kubernetes Engine (GKE) and Compute Engine. Meeting these requirements optimizes database performance on the cloud infrastructure.

#### GKE-based deployment

A GKE-based deployment has the following requirements:

  - Nodes with at least 4 vCPU and at least 16 GB RAM

  - Zonal persistent disk ( `pd-ssd` ) or Hyperdisk Balanced for storage

#### VM-based deployment on Google Cloud

A VM-based deployment on Google Cloud requires a VM with 4 vCPUs and 16 GB RAM. For storage, use a zonal persistent disk ( `pd-ssd` ) or Hyperdisk Balanced.

### Amazon Web Services (AWS)

To deploy Spanner Omni on AWS, follow the Amazon Elastic Kubernetes Service (EKS) and Amazon Elastic Compute Cloud (EC2) configuration requirements. These requirements ensure compatibility with AWS-specific features, such as `/dev/vmclock0` for precise timekeeping.

All AWS deployments must access the `/dev/vmclock0` device. To support this, do the following:

1.  Configure your environment with a supported machine type (for example, `M7a` ) and Amazon Linux 2023.

2.  Enable read permissions on the host by running `sudo chmod a+r /dev/vmclock0` .

3.  Ensure the application can access the device:
    
      - For Docker, pass the device using `--device /dev/vmclock0` .
    
      - For Kubernetes, mount the device path into the Pod specification.

#### EKS-based deployment

An EKS-based deployment requires nodes with 4 vCPU and 16 GB. For storage, use SSD-backed EBS (Zonal, `io2` Block Express or `gp3` ) volumes.

#### VM-based deployment on AWS

A VM-based deployment on AWS requires a minimum of 4 vCPUs and 16 GB RAM. For storage, use SSD-backed EBS (Zonal, `io2` Block Express or `gp3` ) volumes.
