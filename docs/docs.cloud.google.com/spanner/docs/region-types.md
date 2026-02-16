This page describes the different region types available in Spanner:

  - Read-write regions
  - Read-only regions
  - Witness regions

## Read-write regions

Each [dual-region configuration](/spanner/docs/instance-configurations#dual-region-configurations) has two read-write regions, each of which contains two read-write replicas and one witness replica. The read-write replicas behave similarly to the read-write replicas of a multi-region configuration.

Each [multi-region configuration](/spanner/docs/instance-configurations#multi-region-configurations) contains two read-write regions, each of which contains two [read-write replicas](/spanner/docs/replication#read-write) .

One of these read-write regions is designated the default *leader region* . A leader is selected from the replicas in the default leader region for each split. In the event of a leader replica failure, the other replica in the default leader region automatically assumes leadership. In fact, leaders run health checks on themselves and can preemptively give up leadership if they detect they are unhealthy. In most cases, when the default leader region returns to a healthy state, it automatically re-assumes the leadership.

Writes are first processed in the default leader region. You can monitor the percentage of replicas within a given region by using the `  instance/leader_percentage_by_region  ` monitoring metric. For more information, see [Spanner metrics](/monitoring/api/metrics_gcp_p_z#gcp-spanner) .

The second read-write region contains additional replicas that serve reads and participate in voting to commit writes. These additional replicas in the second read-write region are eligible to be leaders. In the unlikely event of the loss of all replicas in the default leader region, new leader replicas are chosen from the second read-write region.

You can configure the leader region of a database by following the instructions at [Change the leader region of a database](/spanner/docs/modifying-leader-region#change-leader-region) . For more information, see [Configure the default leader region](/spanner/docs/instance-configurations#configure-leader-region) .

Optionally, you can grant non-leader, read-write regions the [read lease region](/spanner/docs/read-lease) status. Read lease regions help your database reduce [strong read](/spanner/docs/reads#read_types) latency in dual-region or multi-region instances. However, writes experience higher latency when you use read lease.

**Key Point:** Place most of your read and write workloads in the default leader region. In the event of the loss of a default leader region, read and write workloads are served from the second read-write region.

## Read-only regions

Read-only regions contain [read-only replicas](/spanner/docs/replication#read-only) , which can serve low-latency reads to clients that are outside of the read-write regions. Read-only replicas maintain a full copy of your data, which is replicated from read-write replicas. They don't participate in voting to commit writes and don't contribute to any write latency.

Some base multi-region configurations contain read-only replicas. You can also create a custom instance configuration, and add read-only replicas to your custom regional and multi-region instance configurations to scale reads and support low latency stale reads. All read-only replicas are subject to [compute capacity and database storage costs](/spanner/pricing) .

Furthermore, adding read-only replicas to an instance configuration doesn't change the [Spanner SLAs](/spanner/sla) of the instance configuration. For more information, see [Read-only replicas](/spanner/docs/replication#read-only) .

Optionally, to reduce read latency for transactions that require strong consistency, you can grant non-leader, read-only regions the [read lease region](/spanner/docs/read-lease) status. Read lease regions help your database reduce [strong read](/spanner/docs/reads#read_types) latency in dual-region or multi-region instances. However, writes experience higher latency when you use read lease.

**Key Point:** Place additional read workloads in read-only regions to reduce latency on read-write regions.

## Witness regions

A witness region contains a [witness replica](/spanner/docs/replication#witness) , which is used to form a write quorum and vote on writes. Every Spanner mutation requires a write quorum that's composed of a majority of voting replicas (for dual-region configurations, the quorum requires two replicas from both regions). Witnesses become important in the rare event that the read-write regions become unavailable. Only dual-region and multi-region configurations contain witness regions. For more information about leader regions and voting replicas, see [Replication](/spanner/docs/replication) .

**Key Point:** The witness region are system-configured region for voting and achieving write quorums.

## What's next

  - Learn more about [Regional, dual-region, and multi-region configurations](/spanner/docs/instance-configurations) .
  - Learn more about [Replication](/spanner/docs/replication) .
  - Learn more about [Google Cloud geography and regions](/docs/geography-and-regions) .
