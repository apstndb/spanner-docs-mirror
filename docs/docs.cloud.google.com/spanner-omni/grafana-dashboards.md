---
name: documents/docs.cloud.google.com/spanner-omni/grafana-dashboards
uri: https://docs.cloud.google.com/spanner-omni/grafana-dashboards
title: Use Grafana dashboards to monitor Spanner Omni
description: Learn about the Grafana dashboards and charts available to monitor the health and performance of your Spanner Omni deployments.
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

Monitor the health and performance of your Spanner Omni deployments with Grafana dashboards. These dashboards visualize Spanner Omni metrics ingested into Prometheus, providing comprehensive insights into your deployment's operational state. You gain visibility into overall system health, resource consumption, and critical internal processes.

## Dashboard inventory

The following table provides a high-level summary of the available dashboards:

| Dashboard                 | Key metrics                                         | Primary purpose                                                                                                                                  |
| ------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Overview                  | `QPS` , latency, throughput                         | Monitor high-level deployment performance, including query per second ( `QPS` ), request latency, and data throughput.                           |
| System insights           | `CPU` , memory, lock wait time                      | Focus on database-level resource consumption and health (for example, `CPU` , memory, and lock wait time) for selected databases.                |
| Deployment insights       | `CPU` , memory, and network utilization             | Provide detailed insights into overall deployment resource consumption and network statistics.                                                   |
| Spanner Omni file system  | File operations, latency, and throughput            | Monitor underlying file system operations, performance, latency, and throughput.                                                                 |
| gRPC                      | `RPC` count, status, and latency                    | Track detailed `RPC` statistics for server-side and client-side communication.                                                                   |
| Compactions               | Compaction success and failure rate, compaction lag | Visualize background data maintenance performance, focusing on compaction success and failure rates and compaction lag.                          |
| Splits, merges, and moves | Split, merge, and move count; group size            | Monitor the dynamic distribution of data, including directory operations (splits, merges, and moves), data group sizing, and potential hotspots. |
| Tablets                   | Tablet count, load distribution                     | Provide deep insights into tablet statistics, operations, and load distribution, and identify potential hotspots.                                |
| TrueTime                  | Drift, uncertainty, SLA violations                  | Monitor the health and reliability of the Spanner Omni TrueTime service, including drift, uncertainty, and SLA violations.                       |
| Shared log                | Write rate, sort error rate                         | Monitor shared log performance, specifically write rates and sorting error rates.                                                                |

The following are some of the charts available in the dashboards:

  - **Compute Capacity (vCPU)** : The total number of `vCPUs` provisioned across the deployment.

  - **Memory Capacity** : The total physical memory provisioned across the deployment.

  - **Storage Capacity** : The total and available file system storage capacity.

  - **Node Status** : Tracks the count of total and unhealthy servers in the deployment.

  - **Compute Utilization** : The percentage of total `vCPU` capacity in use.

  - **Memory Utilization** : Total memory usage across the deployment. Expect high usage because Spanner Omni uses idle memory for cache purposes.

  - **Storage Utilization** : The percentage of total storage capacity in use.

  - **Storage Used Per vCPU** : The ratio of total storage used relative to the total `vCPU` count.

  - **Servers** : A detailed table showing per-server and per-zone metrics for `CPU` utilization, memory utilization, uptime, total `vCPU` , total memory, storage used, and storage capacity.

## System insights dashboard

The system insights dashboard focuses on the health and performance of the databases in the deployment. This dashboard includes the following charts:

  - **CPU Utilization Overview** : Database overall `CPU` utilization aggregated across selected servers.

  - **CPU utilization by user and system** : Utilization of `CPU` in the selected database, grouped by user and system tasks and priority.

  - **CPU Utilization By Operation Type** : Utilization of `CPU` grouped by operation type for the selected database aggregated across the selected servers.

  - **CPU Utilization By Operation Type - High Priority** : Utilization of `CPU` grouped by operation type and filtered by high priority for the selected database aggregated across the selected servers.

  - **CPU Utilization By Operation Type - Medium Priority** : Utilization of `CPU` grouped by operation type and filtered by medium priority for the selected database aggregated across the selected servers.

  - **CPU Utilization By Operation Type - Low Priority** : Utilization of `CPU` grouped by operation type and filtered by low priority for the selected database aggregated across the selected servers.

  - **Request Latency** ( `P50` , `P90` , `P99` ): Latency within a selected database, grouped by read and write methods across the selected servers.

  - **Request Latency By Method** ( `P50` , `P90` , `P99` ): Latency within a selected database, grouped by API methods across the selected servers.

  - **Transaction Latency** ( `P50` , `P90` , `P99` ): Request latency within a selected database, grouped by transaction type and leader involvement across the selected servers.

  - **Throughput** : Read and write throughput within a selected database across selected servers.

  - **Throughput By Method** : Throughput within a selected database grouped by method across selected servers.

  - **Operations Per Second** : Operations per second within a selected database grouped by read and write methods across selected servers.

  - **Operations Per Second By Method** : Operations per second within a selected database grouped by methods across selected servers.

  - **Storage Utilization By Database** : Non-replicated physical bytes used by each database. The leader tablet of each group provides this metric. The actual replicated physical bytes across all tablets of a group might be higher or lower depending on the state of compactions on each tablet, but this metric provides an approximate idea of how much non-replicated physical storage each database uses.

  - **Lock Wait Time** : Total lock wait time for lock conflicts for the selected database in a 5-minute interval.

  - **Aborted Transactions Rate** : The rate of aborted, or canceled, transactions. Cancelation rates can be higher when conflicts occur between transactions.

  - **Schema Object Count** : Number of schema objects for the selected database.

  - **Transaction Participants** : Distribution of the number of transaction participants in each commit attempt for the database.

## Deployment insights dashboard

The deployment insights dashboard provides further insights into deployment resource consumption. This dashboard includes the following charts:

  - **CPU utilization** : Aggregated `CPU` utilization for the selected servers.

  - **Server CPU utilization** : `CPU` utilization for each selected server.

  - **Process CPU utilization** : `CPU` utilization for each process aggregated across the selected servers.

  - **Memory utilization** : Aggregated memory utilization for the selected servers. Expect high values because Spanner Omni caches data in memory, which Spanner Omni can free if needed.

  - **Server Memory utilization** : Memory utilization for each of the selected servers. Expect high values because Spanner Omni caches data in memory, which Spanner Omni can free if needed.

  - **Process Resident Memory Size** : Resident memory size for each process for selected servers.

  - **Process Virtual Memory Size** : Virtual memory size for each process for selected servers.

  - **Server Memory Breakdown** : Memory utilization by category (cache, fragmented, `memtable_pinned` , system, updates, other) aggregated across the selected servers. This memory is specific to the `span_server` process.

  - **Network Sent Bytes** : Sent bytes per interface aggregated across all servers.

  - **Network Received Bytes** : Received bytes per interface aggregated across all servers.

  - **Top 10 Servers By Network Sent Bytes** : Top 10 servers by network sent bytes (table view).

  - **Top 10 Servers By Network Received Bytes** : Top 10 servers by network received bytes (table view).

## Spanner Omni file system dashboard

The Spanner Omni file system dashboard monitors the underlying file system operations critical to performance, including operation rates, latency, and throughput. This dashboard includes the following charts:

  - **File Operation Graphs** :
    
      - **Operations Per Second** : Tracks the total rate of file operations, grouped by operation.
    
      - **Local and remote operations per second** : Tracks the rate of file operations, separated by local versus remote access.
    
      - **Operation Errors Per Second** : Shows the rate of failed file system operations, grouped by operation and status.

  - **Latency Graphs** : Includes charts for `P50` , `P90` , and `P99` latency for local and remote file operations, grouped by operation.

  - **Throughput Graphs** :
    
      - **Local and Remote Read and Write Throughput** : Tracks the rate of read and write throughput, separated by local and remote access.
    
      - **Bytes Per Operation** : Includes `P50` and `P90` bytes transferred per operation for local and remote access.

  - **File system Statistics** :
    
      - **Total File system Size By Zone** : Displays the total provisioned file system size, grouped by Spanner Omni zone.
    
      - **File system Usage by Zone** : Displays the current used file system size, grouped by Spanner Omni zone.

## gRPC dashboard

The gRPC dashboard tracks detailed `RPC` statistics for all servers within the deployment. This dashboard includes the following charts:

  - **Server Side Metrics** : Monitors `RPC` performance from the server's perspective.
    
      - **RPC Latency Per Method** ( `P50` , `P90` , `P99` ): Latency per `RPC` method on the server side.
    
      - **Server Sent Throughput Per Method** : Sent bytes per second per method for selected servers.
    
      - **Server Sent Throughput Per Process** : Sent bytes per second per process for selected servers.
    
      - **Server Received Throughput Per Method** : Received bytes per second per method for selected servers.
    
      - **Server Received Throughput Per Process** : Received bytes per second per process for selected servers.
    
      - **Server Canonical Status Count Per Method** : Rate of occurrence of canonical status code per method for selected servers.
    
      - **Server Completed RPCs Per Method** : Rate of completed `RPCs` per method for selected servers.
    
      - **Server Active Channels** : The total number of server-side `gRPC` channels created since the application started that remain active.

  - **Client Side Metrics** : Monitors `RPC` performance from the client's perspective.
    
      - **Client Roundtrip Latency Per Method** ( `P50` , `P90` , `P99` ): Roundtrip `RPC` latency per method, which includes server latency, network, and queue time.
    
      - **Client Sent Throughput Per Method** : Sent bytes per second per method for selected servers.
    
      - **Client Sent Throughput Per Process** : Sent bytes per second per process for selected servers.
    
      - **Client Received Throughput Per Method** : Received bytes per second per method for selected servers.
    
      - **Client Received Throughput Per Process** : Received bytes per second per process for selected servers.
    
      - **Client Canonical Status Count Per Method** : Rate of occurrence of canonical status code per method as a `gRPC` client for selected servers.
    
      - **Client Completed RPCs Per Method** : Rate of completed client `RPCs` per method for selected servers.

## Compactions dashboard

The compactions dashboard shows a visualization of the performance of background compaction tasks. This dashboard includes the following charts:

  - **Successful and Failed Compactions (Last 1h)** : Tracks the successful and failed counts of compaction types grouped by compaction type and per server.

  - **Compactions Output Bytes Rate** : Tracks the output bytes rate of compactions over a 2-minute interval, grouped by compaction type and per server.

  - **Compactions Input Size Distribution** : A heatmap shows the distribution of compaction input sizes.

  - **Compactions Input Size (Mean)** : Displays the mean compaction input size, grouped by compaction type and per server.

  - **Compactions Input Size (Percentile Estimates)** : Provides percentile estimates ( `P50` , `P95` , `P99` ) of compaction input size, grouped by compaction type and per server.

  - **Major Compaction Lag Distribution** : A heatmap shows the distribution of major compaction lag aggregated across all servers.

  - **Major Compaction Lag (Mean) Per Server** : Shows the mean of major compaction lag per server.

  - **Major Compaction Lag (Percentile Estimates) Per Server** : Provides percentile estimates ( `P50` , `P90` , `P99` ) of major compaction lag per server.

## Splits, merges, and moves dashboard

The splits, merges, and moves dashboard tracks the dynamic distribution of data across the cluster, including directory operations and group sizing. This dashboard includes the following charts:

  - **Split Size Distribution** : The size of the directory split, including `P50` , `P90` , `P99` , and `P100` percentiles, aggregated across selected servers.

  - **Group Size Distribution** : All bytes allocated for the group (persistent and in-memory), with `P50` , `P90` , `P99` , and `P100` percentiles, aggregated across selected servers.

  - **Group Size In Memory Distribution** : All bytes allocated for the group in-memory data structures, with `P50` , `P90` , `P99` , and `P100` percentiles, aggregated across selected servers.

  - **Group Size By Zone** : The `P50` , `P90` , `P99` , and `P100` sizes for all allocated bytes (persistent and in-memory) for the group, grouped by Spanner Omni zone.

  - **Number of successful internal data moves** : Counts of directory and group moves, splits, and merges over a 1-hour window, grouped by initiator, action, and move type.

  - **Number of failed internal data moves** : Counts of errors in attempted directory and group moves, splits, and merges over a 1-hour window.

  - **Unsplittable errors by reason and type** : Rate of unsplittable errors where overloaded splits are ignored because the range was unsplittable.

  - **Peak Split CPU Usage Score** : The maximum `CPU` usage load across all splits of each database.

## Tablets dashboard

The tablets dashboard provides deep insights into tablet statistics, operations, and potential hotspots. This dashboard includes the following charts:

  - **Total Tablet Count** : The total number of Paxos tablets across the deployment.

  - **Tablet Count By Zone** : The number of tablets, grouped by Spanner Omni zone.

  - **Tablet Count By Server** : The number of tablets on selected servers.

  - **Leader Count By Zone** : The count of leader tablets, grouped by Spanner Omni zone.

  - **Leader Count By Server** : The count of leader tablets on selected servers.

  - **Unassigned Tablets Per Zone** : The number of unassigned tablets per zone.

  - **Tablet Loads By Zone** : The rate of tablet loads grouped by zone.

  - **Tablet Unloads By Zone By Reason** : The rate of tablet unloads per zone, categorized by the reason for the unload.

  - **Max Tablet Load For Each Server** : A table view displays the maximum compute load for a tablet on each server.

  - **Hot Tablets Count** : The total count of hot tablets (tablets exceeding a compute load threshold).

  - **Tablet Load Distribution** : The distribution of per-tablet compute load, displaying `P50` and `P90` percentile estimates, and the exact `MAX` value.

## TrueTime dashboard

The TrueTime dashboard gives visibility into the health and reliability of the Spanner Omni TrueTime service. This dashboard includes the following charts:

  - **TrueTime availability** : Monitors the overall availability of the TrueTime service.

  - **P99 TrueTime drift** : Tracks the 99th percentile of TrueTime drift.

  - **P99 TrueTime uncertainty** : Tracks the 99th percentile of TrueTime uncertainty.

  - **Clock SLA violations** : Shows the count of violations against the clock service-level agreement (SLA).

  - **VM migration counts** : Tracks the number of virtual machine migrations.

  - **TrueTime drift on leader** : Monitors the TrueTime drift specifically on the leader nodes.

  - **TrueTime desired and actual steering ppm** : Compares the desired and actual parts-per-million ( `ppm` ) steering values.

  - **TrueTime steering error** : Tracks the error in the TrueTime steering mechanism.

## Shared log dashboard

The shared log dashboard is a dedicated dashboard for monitoring shared log performance and recovery status. This dashboard includes the following charts:

  - **Shared Log Write Rate** : Shared log entry count per second, aggregated and broken down per database.

  - **Shared Log Bytes Written** : Shared log bytes written per second (throughput), aggregated and broken down per database.

  - **Shared Log Batches Write Rate** : Shared log batches written per second aggregated across selected servers.

  - **Shared Log Batch Write Latency Distribution** : The `P50` , `P90` , and `P99` latency distribution for shared log batch writes.

  - **Shared Log Batch Entries Count Distribution** : The `P50` , `P90` , and `P99` distribution of entry counts within shared log batches.

  - **LogSort Request Rate** : The rate of `LogSort` requests aggregated across selected servers.

  - **LogSort Sorting Error Rate** : The rate of `LogSort` sorting errors aggregated across selected servers.

  - **In Progress Shared Log Readers** : The total number of shared log readers that are engaged in tablet recovery.

## What's next

  - [Use Prometheus alerts to monitor Spanner Omni](https://docs.cloud.google.com/spanner-omni/prometheus-alerts) .

  - [Add encryption to a Spanner Omni Kubernetes deployment](https://docs.cloud.google.com/spanner-omni/deploy-encryption-kubernetes) .

  - [Add encryption to a Spanner Omni VM deployment](https://docs.cloud.google.com/spanner-omni/deploy-encryption-vms) .
