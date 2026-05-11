---
name: documents/docs.cloud.google.com/spanner-omni/key-terms
uri: https://docs.cloud.google.com/spanner-omni/key-terms
title: Spanner Omni key terms
description: Understand Spanner Omni key terms and concepts.
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document defines the core concepts and deployment topologies for Spanner Omni. It covers the hierarchical relationship between regions, zones, and servers, and explains how these components relate to data replication and storage within a deployment.

## Spanner Omni concepts

The following sections provide detailed definitions of the terminologies used throughout the Spanner Omni documentation. Familiarizing yourself with these concepts ensures a clear understanding of the system's architecture and operational behavior.

### Deployment

A deployment of Spanner Omni, which lets you use the databases in your data centers or your public cloud accounts. You create the deployment according to the parameters specified in deployment configuration. A Spanner Omni deployment is equivalent to a [Spanner| instance](https://docs.cloud.google.com/spanner/docs/instances) in Google Cloud.

### Deployment configuration

Provides the placement and specification of regions, zones, and servers for your Spanner Omni deployment. You can choose a single server, single region, or multi-region deployment configuration.

### Location

The equivalent of region in Google Cloud terms. The network latency between two regions is non-trivial. For on-premises deployments, you can define your own regions. For cloud deployments, the region must align with the regions in the respective cloud provider. One region can have multiple zones.

### Process

The Spanner server forks and manages multiple processes. For example, TimeServer, spanserver are all processes in Spanner. Individual processes may have monitoring statistics like CPU and memory usage. A process can open ports to communicate with other servers in the deployment.

### Replica

Spanner replicates data to provide data availability and geographic locality. At a high level, Spanner organizes all data into rows. Spanner creates multiple copies, or replicas of these rows, then stores these replicas in different geographic areas. Spanner uses a synchronous, Paxos-based replication scheme, in which voting replicas vote on every write request before it commits the write to the database. Like in Spanner, there are three replica types in Spanner Omni: *read-write* , *read-only* , and *witness* . For more information, see [replica types](https://docs.cloud.google.com/spanner/docs/replication#replica-types) in Spanner documentation.

### Server

A server is a compute resource, such as a VM or container, where the Spanner Omni server runs. Each server has its own system resources: CPU, memory, and storage. The server provides Spanner Omni's capabilities by storing and serving user data.

#### Root servers

Root servers store critical metadata to support the zone. For example, the root server stores server membership and other zone configuration information. Root servers use quorum algorithms for consistency, so there must be an odd number of Root Servers in a zone: one for very small zones, and three to five for larger zones. Consider the number of root servers carefully while planning the deployment. While you can change the number of root servers in the deployment after you create it, we don't recommend it.

#### Non-root servers

Non-root servers store and serve user data and provide a way to scale the compute capacity and storage of a zone. You can add as many non-root servers to your deployment as the workload demands. You can change the number of non-root servers after you create the deployment.

### Split

A Spanner split holds a range of contiguous rows of data, where Spanner orders the rows by primary key. Spanner creates replicas of each split that it stores in each zone.

### Storage

The permanent storage attached to the server.

### Zone

A group of one or more servers. For data replication, you should create one zone per replica. For on-premises deployments, we recommend minimizing infrastructure sharing (VMs, disks) between zones. For cloud deployments, align zones to the availability zones in AWS, or zones in Google Cloud.
