This page describes how data is replicated in Spanner, the different types of Spanner replicas and their roles in reads and writes, and the benefits of replication.

## Overview

Spanner automatically replicates at the byte level. As described in [Life of Spanner Reads and Writes](/spanner/docs/whitepapers/life-of-reads-and-writes#aside-distributed-filesystems) , it takes advantage of this capability in the underlying file system that it's built on. Spanner writes database mutations to files in this file system, and the file system takes care of replicating and recovering the files when a machine or disk fails.

Even though the underlying distributed file system that Spanner is built on already provides byte-level replication, Spanner also replicates data to provide the additional benefits of data availability and geographic locality. At a high level, all data in Spanner is organized into rows. Spanner creates multiple copies, or *replicas* , of these rows, then stores these replicas in different geographic areas. Spanner uses a synchronous, Paxos-based replication scheme, in which [voting replicas](#replica-types) take a vote on every write request before the write is committed. This property of globally synchronous replication lets you read the most up-to-date data from any Spanner read-write or read-only replica.

Spanner creates replicas of each database [split](/spanner/docs/schema-and-data-model#database-splits) . A *split* holds a range of contiguous rows, where the rows are ordered by primary key. All of the data in a split is physically stored together in the replica, and Spanner serves each replica out of an independent failure zone. For more information, see the [Schemas overview](/spanner/docs/schema-and-data-model) .

A set of splits is stored and replicated using [Paxos](https://en.wikipedia.org/wiki/Paxos_\(computer_science\)) . Within each Paxos replica set, one replica is elected to act as the *leader* . Leader replicas handle writes, while read-write or read-only replicas can serve a read request without communicating with the leader. If a strong read is requested, the leader typically is consulted to ensure that the read-only replica has received all recent mutations. To monitor the rate of change and amount of data that is replicated from your leader replica to the cross region replicas in your instance configuration, see [Monitor data replication](#monitor-replication) .

## Benefits of Spanner replication

The benefits of Spanner replication include:

  - **Data availability** : Having more copies of your data makes the data more available to clients that want to read it. Also, Spanner can still serve writes even if some of the replicas are unavailable, because only a majority of voting replicas are required in order to commit a write.

  - **Geographic locality** : Having the ability to place data across different regions and continents with Spanner means data can be geographically closer, and hence faster to access, to the users and services that need it.

  - **Single database experience** : Spanner can deliver a single database experience because of its synchronous replication and global strong consistency.

  - **Easier application development** : Because Spanner is ACID-compliant and offers global strong consistency, developers working with Spanner don't have to add extra logic in their applications to deal with eventual consistency, making application development and subsequent maintenance faster and easier.

## Replica types

Spanner has three types of replicas: *read-write replicas* , *read-only replicas* , and *witness replicas* . The regions and replication topologies that form [base instance configurations](/spanner/docs/instance-configurations) are fixed:

  - [Base single-region (regional) instance configurations](/spanner/docs/instance-configurations#available-configurations-regional) only use read-write replicas.
  - [Base dual-region instance configurations](/spanner/docs/instance-configurations#available-configurations-dual-region) use read-write and witness replicas.
  - [Base multi-region instance configurations](/spanner/docs/instance-configurations#available-configurations-multi-region) use a combination of all three replica types.

You can create custom instance configurations and add additional [read-only replicas](#read-only) for regional and multi-region instance configurations.

The following table summarizes the types of Spanner replicas and their properties:

<table>
<thead>
<tr class="header">
<th>Replica type</th>
<th>Can vote</th>
<th>Can become leader</th>
<th>Can serve reads</th>
<th>Can configure replica manually</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><strong>Read-write</strong></td>
<td>yes</td>
<td>yes</td>
<td>yes</td>
<td>no</td>
</tr>
<tr class="even">
<td><strong>Read-only</strong></td>
<td>no</td>
<td>no</td>
<td>yes</td>
<td>yes <sup>*</sup></td>
</tr>
<tr class="odd">
<td><strong>Witness</strong></td>
<td>yes</td>
<td>no</td>
<td>no</td>
<td>no</td>
</tr>
</tbody>
</table>

<sup>\*</sup> For more information, see how to [create an instance with a custom instance configuration](/spanner/docs/create-manage-configurations#create-configuration) .

### Read-write replicas

Read-write replicas support both reads and writes. These replicas:

  - Maintain a full copy of your data.
  - Serve reads.
  - Can vote whether to commit a write.
  - Participate in leadership election.
  - Are eligible to become a leader.
  - Are the only replica type used in regional instances.

### Read-only replicas

Read-only replicas only support reads, but not writes. These replicas don't vote for leaders or for committing writes, so they allow you to scale your read capacity without increasing the quorum size needed for writes. Read-only replicas:

  - Maintain a full copy of your data, which is replicated from the leader read-write replica.

  - Don't participate in voting to commit writes. Hence, the location of the read-only replicas never contributes to write latency.

  - Aren't eligible to become a leader.

  - Serve reads.

  - Can scale asymmetrically. For more information, see [Asymmetric read-only autoscaling](/spanner/docs/managed-autoscaler#asymmetric-read-only-autoscaling) .

  - If it is the nearest replica to your application, the read-only replica can usually serve stale reads without needing a round trip to the leader region, assuming staleness is at least 15 seconds. You can also use directed reads to route read-only transactions and single reads to a specific replica type or a region in a multi-region instance configuration. For more information, see [Directed reads](/spanner/docs/directed-reads) .
    
    Strong reads might require a round trip to the leader replica. The round trip is just for negotiating the timestamp, not shipping the actual data from the leader. The timestamp negotiation is a CPU efficient operation at the leader, and typically the data is already en-route. This communication is handled automatically by the system.
    
    For more information about stale and strong reads, see the [In reads section](#role-in-reads) .

#### Optional read-only replicas

You can [create a custom regional or multi-region instance configuration](/spanner/docs/create-manage-configurations#create-configuration) and add optional read-only replicas to scale reads and support low latency stale reads. The added read-only replica must be in a region that isn't part of the predefined base instance configuration. For a list of the optional read-only regions that you can add, see the Optional Region column under [Regional available configurations](/spanner/docs/instance-configurations#available-configurations-regional) and [Multi-region available configurations](/spanner/docs/instance-configurations#available-configurations-multi-region) . If you don't see your chosen read-only replica location, you can [request a new optional read-only replica region](https://docs.google.com/forms/d/e/1FAIpQLSfw9Rj4p4KA8oLu7MhIpSyPRd-4qxqazwsFZIY-_tkNrpWFcw/viewform) .

All optional read-only replicas are subject to [compute capacity, storage, and replication costs](https://cloud.google.com/spanner/pricing) .

Furthermore, adding read-only replicas to a custom instance configuration doesn't change the [Spanner SLAs](https://cloud.google.com/spanner/sla) of the instance configuration.

If you choose to add a read-only replica to a continent that's in a different continent than the leader region, we recommend adding a minimum of two read-only replicas. This helps maintain low read latency in the event that one of the read-only replicas becomes unavailable.

As a best practice, test performance workloads in non-production instances in the custom instance configuration first. You can refer to the [Inter-Region Latency and Throughput benchmark dashboard](https://datastudio.google.com/s/veoams1Brls) for median inter-region latency data. For example, if you create a custom instance configuration with the `  eur6  ` multi-region base configuration and an optional read-only replica in `  us-east1  ` , the expected strong read latency for a client in `  us-east1  ` is about 100 milliseconds due to the round trip time to the leader region in `  europe-west4  ` . Stale reads with sufficient staleness don't incur the round trip and are therefore much faster. You can also use the lock insights and transaction insights to [identify transactions that lead to high latencies](/spanner/docs/use-lock-and-transaction-insights) .

For instructions on how to add optional read-only replicas, see [Create a custom instance configuration](/spanner/docs/create-manage-configurations#create-configuration) .

### Witness replicas

Witness replicas don't support reads but do participate in voting to commit writes. These replicas make it easier to achieve quorums for writes without the storage and compute resources that are required by read-write replicas to store a full copy of data and serve reads. Witness replicas:

  - Are used in dual-region and multi-region instances.
  - Don't maintain a full copy of data.
  - Don't serve reads.
  - Vote whether to commit writes.
  - Participate in leader election but aren't eligible to become a leader replica.

## The role of replicas in writes and reads

This section describes the role of replicas in Spanner writes and reads, which is helpful in understanding why Spanner uses witness replicas in dual-region and multi-region configurations.

### In writes

**Note:** In Spanner, deletes are a type of write.

Client write requests are always processed at the leader replica first, even if there is a non-leader replica that's closer to the client, or if the leader replica is geographically distant from the client. If you use a dual-region or multi-region instance configuration and your client application is located in a non-leader region, Spanner uses leader-aware routing to route read-write transactions dynamically to reduce latency in your database. For more information, see [Leader-aware routing](/spanner/docs/leader-aware-routing) .

The leader replica logs the incoming write, and forwards it, in parallel, to the other replicas that are eligible to vote on that write. Each eligible replica completes its write, and then responds back to the leader with a vote on whether the write should be committed. The write is committed when a majority of voting replicas (or *write quorum* ) agree to commit the write. In the background, all remaining (non-witness) replicas log the write. If a read-write or read-only replica falls behind on logging writes, it can request the missing data from another replica to have a full, up-to-date copy of the data.

### In reads

Client read requests might be executed at or require communicating with the leader replica, depending on the concurrency mode of the read request.

  - Reads that are part of a [read-write transaction](/spanner/docs/transactions#read-write_transactions) are served from the leader replica, because the leader replica maintains the locks required to enforce serializability.

  - Single read methods (a read outside the context of a transaction) and reads in [read-only transactions](/spanner/docs/transactions#read-only_transactions) might require communicating with the leader, depending on the concurrency mode of the read. For more information about concurrency modes, see [Read types](/spanner/docs/reads#read_types) .
    
      - Strong read requests can go to any read-write or read-only replica. If the request goes to a non-leader replica, that replica must communicate with the leader in order to execute the read.
    
      - Stale read requests go to the closest available read-only or read-write replica that's caught up to the timestamp of the request. This can be the leader replica if the leader is the closest replica to the client that issued the read request.

### Aside: Why read-only and witness replicas?

[Base multi-region configurations](/spanner/docs/instance-configurations#available-configurations-multi-region) use a combination of read-write, read-only, and witness replicas, whereas [base dual-region configurations](/spanner/docs/instance-configurations#available-configurations-dual-region) use read-write and witness replicas, and [base regional configurations](/spanner/docs/instance-configurations#available-configurations-regional) use only read-write replicas. The reasons for this difference have to do with the varying roles of replicas in writes and reads. For writes, Spanner needs a majority of voting replicas to agree on a commit in order to commit a mutation. In other words, every write to a Spanner database requires communication between voting replicas. To minimize the latency of this communication, it is desirable to use the fewest number of voting replicas, and to place these replicas as close together as possible. That's why base regional configurations contain exactly three read-write replicas, each of which is in its own availability zone, contains a full copy of your data, and is able to vote. If one replica fails, the other two can still form a write quorum, and because replicas in this configuration are in the same geographic region, network latencies are minimal.

Base dual-region and multi-region configurations contain more replicas by design, and these replicas are in different data centers (so that clients can read their data quickly from more locations). What characteristics should these additional replicas have? They could all be read-write replicas, but that would be undesirable because adding more read-write replicas to a configuration increases the size of the write quorum (which means potentially higher network latencies due to more replicas communicating with each other, especially if the replicas are in geographically distributed locations) and also increases the amount of storage needed (because read-write replicas contain a full copy of data). Instead of using more read-write replicas, base dual-region configuration contain an additional witness replica, and base multi-region configurations contain read-only replicas and witness replicas, which have fewer responsibilities than read-write replicas.

  - Read-only replicas don't vote for leaders or for committing writes, so they allow you to scale your read capacity without increasing the quorum size needed for writes.
  - Witness replicas vote for leaders and for committing writes, but don't store a full copy of the data, can't become the leader, and can't serve reads. They make it easier to achieve quorums for writes without the storage and compute resources that are required by read-write replicas to store a full copy of data and serve reads.

## Monitor data replication

You can monitor the rate of change and the amount of data that's replicated from your leader replica to the cross region replicas in your instance configuration. The rate of change is in bytes per second and the amount of data is in bytes. To do this, use the [Cross region replicated bytes](/monitoring/api/metrics_gcp_p_z#gcp-spanner) ( `  instance/cross_region_replicated_bytes_count  ` ) monitoring metric.

**Note:** The metric is sampled every 60 seconds. After sampling, data isn't visible for up to 120 seconds. For more information, see [Latency of metric data](/monitoring/api/v3/latency-n-retention#latency) .

To view this metric in the Google Cloud console, follow these steps:

1.  In the Google Cloud console, go to **Monitoring** :

2.  In the navigation menu, select **Metrics explorer** .

3.  In the **Metric** field, click the **Select a metric** drop-down.

4.  In the **Filter by resource or metric name** field, select **Cloud Spanner Instance \> Instance \> Cross region replicated bytes** , and then click **Apply** .
    
    This metric is available only under **Active metrics** if there is cross region replication activity in your instance. Otherwise, it appears under **Inactive metrics** . By default, the UI filters and shows only Active metrics. Clear the **Active** checkmark to view both active and inactive metrics.
    
    The chart shows the rate of change (in bytes per second) of replicated data across all Spanner instances within the specified time range.

5.  Optional: To show the amount of data (in bytes) that's replicated instead of the rate of change:
    
    1.  In the **Aggregation** field, click the **Sum** drop-down, and select **Configure aligner** .
    
    2.  In the **Alignment function** field, click the **Rate** drop-down, and select **Delta** .
    
    3.  Select **Table** or **Both** as the table type instead of Chart.
        
        The table shows the amount of data (in bytes) that was replicated within the specified time range.

6.  Optional: To view usage for a particular instance or attribute:
    
    1.  Use the **Filter** field to add filters, such as an instance ID, database ID, source region, destination region, or a tag.
    2.  Click **Add filter** to add multiple filters.

To see a full list of Google Cloud metrics, see [Google Cloud metrics](/monitoring/api/metrics_gcp) .

## What's next

  - Learn more about [instance configurations](/spanner/docs/instance-configurations) .
  - Learn how to [create and manage instances](/spanner/docs/create-manage-instances) .
  - Learn how to [create and manage instance configurations](/spanner/docs/instance-configurations) .
  - Learn more about [Google Cloud geography and regions](/docs/geography-and-regions) .
