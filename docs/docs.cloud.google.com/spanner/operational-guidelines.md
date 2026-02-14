This page describes some of the user-controlled configurations that can cause an outage for a Spanner instance to be excluded from the Spanner Service Level Agreement (SLA), which excludes outages "caused by factors outside of Google's reasonable control". It also provides guidelines on how to avoid these configurations.

Spanner manages many aspects of database operations, such as splitting and rebalancing data, replication, failover, and all hardware and software updates. You can configure many of these behaviors with built-in settings and administrative APIs. Your workloads also depend on other components in addition to Spanner, such as your applications and network. These customer-controlled configurations may increase the risk of instance downtime, depending on your database load and other configuration parameters.

If your instance becomes unhealthy, and if Google determines that the instance is out of compliance with the operational limits described on this page, then any resulting downtime may not be covered by (or does not count against) the Spanner SLA.

**Note:** You're responsible for monitoring your Spanner instances and verifying that they are correctly sized and configured for the type of workloads that you're running.

## Configurations excluded from the Spanner SLA

The following configurations are excluded from the Spanner SLA:

  - If your instance is configured and used in a way that causes the workload to overload the instance, then it isn't covered by the SLA.
  - Downtime of instances that results from your voluntary actions or inactions isn't covered by the SLA
  - If you disable the [Spanner API](/spanner/docs/getting-started/set-up#set_up_a_project) or other Google Cloud APIs that are required to create and connect to Spanner, then it isn't covered by the SLA.
  - Unavailability of the Spanner API that is the result of your network configuration, such as proxy and firewall rules, isn't covered by the SLA.
  - Application unavailability due to out-of-date or misconfigured [clients](/spanner/docs/reference/libraries) isn't covered by the SLA. In particular, verify that you are using recent client versions with supported dependencies. For example, Java applications should use [Google's BOM](/java/docs/bom) (bill of materials) with a package manager, such as Gradle or Maven.

We recommend that you set up alerts and monitoring using [Cloud Monitoring](/spanner/docs/monitoring-cloud) .

### Configurations to avoid

To maintain Spanner SLA coverage, you must avoid the following configurations:

  - **CPU overload** : If your CPU utilization is consistently high, then your instance isn't properly sized for your workload, and the instance might not be covered by the SLA. Spanner [CPU utilization recommendations](/spanner/docs/compute-capacity#change-compute-capacity) provide overhead for a failover event, where the remaining compute resources help to accommodate traffic from unavailable parts of the instance. You can use Spanner [CPU utilization metrics](/spanner/docs/cpu-utilization) to monitor CPU utilization.
  - **Full storage** : Spanner bills you only for the storage that you use. However, each node, or unit of compute, has a [limit](/spanner/quotas#database-limits) for the amount of storage it can manage. If your instance isn't properly sized for the addressable storage per node, then the instance might not be covered by the SLA. You can use Spanner [storage utilization metrics](/spanner/docs/storage-utilization) to monitor storage utilization.
  - **Quota limit:** Node resources are limited by per-user [quotas](/spanner/quotas) . Failure to request quota increases in advance might result in compute resource overload, which might not be covered by the SLA. Quota increase requests that require approval from Google are typically fulfilled within one day.
  - **Under provisioned sessions** : Spanner clients use [gRPC channels](/spanner/docs/sessions#configure_the_number_of_sessions_and_grpc_channels_in_the_pools) to communicate with Google Cloud endpoints for queries and administration. If your client environments don't provide enough channels to support the request volume of a workload, your applications might experience high latency and low request throughput that might not be covered by the SLA.
  - **Connection overload:** Many Spanner APIs can be safely retried in the event of a transient failure, such as a transaction deadlock in a query, a network issue, or rate limits for administrative APIs. Overly aggressive retries might overwhelm existing connections, causing resource exhaustion or additional throttling. The increased latency or reduced throughput might not be covered by the SLA. For more information, see [managing client timeouts and retries](/spanner/docs/custom-timeout-and-retry) .
  - **Hard disk drive (HDD) overload:** [Tiered storage](/spanner/docs/tiered-storage) lets you store your Spanner data on a mix of solid-state drives (SSD) and hard disk drives (HDD). If your disk load on HDD storage reaches 100%, your Spanner instance experiences significantly increased latency and might not be covered by the SLA. You can use Spanner [tiered storage metrics](/spanner/docs/monitoring-console#tiered_storage_charts_and_metrics) to monitor disk load.

## What's next

  - Learn [best practices for improving Spanner performance and availability using the launch checklist](/spanner/docs/launch-checklist) .
