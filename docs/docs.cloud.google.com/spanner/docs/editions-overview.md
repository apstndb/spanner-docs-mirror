This page describes Spanner editions and its key features.

Spanner editions is a tier-based pricing model that provides different capabilities at different price points. Spanner offers the following editions to support your various business and application needs:

  - **Standard edition** : provides a comprehensive suite of established capabilities that include all of the features that are Generally Available (GA) prior to September 24, 2024 along with selected additional capabilities, such as reverse ETL from BigQuery and scheduled backups, in single-region (regional) instance configurations.

  - **Enterprise edition** : builds on the Standard edition and offers multi-model capabilities including Spanner Graph, full-text search, and Vector Search. It also offers enhanced operational simplicity and data protection using managed autoscaling and incremental backups.

  - **Enterprise Plus edition** : designed for the most demanding workloads that require 99.999% availability with multi-region instance configurations and geo-partitioning support. This tier includes all Standard edition and Enterprise edition features.

## Editions features

The following table lists the features available for each edition.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th></th>
<th>Standard</th>
<th>Enterprise</th>
<th>Enterprise Plus</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="https://cloud.google.com/spanner/sla">Availability SLA</a></td>
<td>99.99% availability SLA</td>
<td>99.99% availability SLA</td>
<td>Up to 99.999% availability SLA</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/instance-configurations">Configurations</a></td>
<td>Regional</td>
<td>Regional<br />
Optional custom read-only replicas</td>
<td>Regional, dual-region, and multi-region<br />
Optional custom read-only replicas</td>
</tr>
<tr class="odd">
<td>Multi-model capabilities</td>
<td>Relational ( <a href="/spanner/docs/reference/standard-sql/overview">GoogleSQL</a> , <a href="/spanner/docs/reference/postgresql/overview">PostgreSQL</a> )<br />
<a href="/spanner/docs/non-relational/overview">Key-value</a></td>
<td>Relational ( <a href="/spanner/docs/reference/standard-sql/overview">GoogleSQL</a> , <a href="/spanner/docs/reference/postgresql/overview">PostgreSQL</a> )<br />
<a href="/spanner/docs/non-relational/overview">Key-value</a><br />
<a href="/spanner/docs/graph/overview">Spanner Graph</a></td>
<td>Relational ( <a href="/spanner/docs/reference/standard-sql/overview">GoogleSQL</a> , <a href="/spanner/docs/reference/postgresql/overview">PostgreSQL</a> )<br />
<a href="/spanner/docs/non-relational/overview">Key-value</a><br />
<a href="/spanner/docs/graph/overview">Spanner Graph</a></td>
</tr>
<tr class="even">
<td>Search capabilities</td>
<td>—</td>
<td><a href="/spanner/docs/full-text-search">Full-text search</a><br />
Vector search ( <a href="/spanner/docs/find-k-nearest-neighbors">KNN</a> , <a href="/spanner/docs/find-approximate-nearest-neighbors">ANN</a> )</td>
<td><a href="/spanner/docs/full-text-search">Full-text search</a><br />
Vector search ( <a href="/spanner/docs/find-k-nearest-neighbors">KNN</a> , <a href="/spanner/docs/find-approximate-nearest-neighbors">ANN</a> )</td>
</tr>
<tr class="odd">
<td>Resource management</td>
<td><a href="/spanner/docs/autoscaler-tool-overview">Open source Autoscaler</a></td>
<td><a href="/spanner/docs/autoscaler-tool-overview">Open source Autoscaler</a><br />
<a href="/spanner/docs/managed-autoscaler">Managed autoscaler</a><br />
<a href="/spanner/docs/managed-autoscaler#asymmetric-read-only-autoscaling">Asymmetric read-only autoscaling</a><br />
<a href="/spanner/docs/create-manage-locality-groups">Locality groups</a><br />
<a href="/spanner/docs/tiered-storage">Tiered storage</a></td>
<td><a href="/spanner/docs/autoscaler-tool-overview">Open source Autoscaler</a><br />
<a href="/spanner/docs/managed-autoscaler">Managed autoscaler</a><br />
<a href="/spanner/docs/managed-autoscaler#asymmetric-read-only-autoscaling">Asymmetric read-only autoscaling</a><br />
<a href="/spanner/docs/create-manage-locality-groups">Locality groups</a><br />
<a href="/spanner/docs/tiered-storage">Tiered storage</a><br />
<a href="/spanner/docs/geo-partitioning">Geo-partitioning</a></td>
</tr>
<tr class="even">
<td>Analytics</td>
<td><a href="/bigquery/docs/spanner-federated-queries">BigQuery federation</a><br />
<a href="/spanner/docs/databoost/databoost-overview">Spanner Data Boost</a><br />
<a href="/bigquery/docs/export-to-spanner">Reverse ETL (BigQuery to Spanner)</a></td>
<td><a href="/spanner/docs/columnar-engine">Columnar engine</a><br />
<a href="/bigquery/docs/spanner-federated-queries">BigQuery federation</a><br />
<a href="/spanner/docs/databoost/databoost-overview">Spanner Data Boost</a><br />
<a href="/bigquery/docs/export-to-spanner">Reverse ETL (BigQuery to Spanner)</a></td>
<td><a href="/spanner/docs/columnar-engine">Columnar engine</a><br />
<a href="/bigquery/docs/spanner-federated-queries">BigQuery federation</a><br />
<a href="/spanner/docs/databoost/databoost-overview">Spanner Data Boost</a><br />
<a href="/bigquery/docs/export-to-spanner">Reverse ETL (BigQuery to Spanner)</a></td>
</tr>
<tr class="odd">
<td>Data protection</td>
<td><a href="/spanner/docs/backup">Standard backups</a><br />
<a href="/spanner/docs/pitr">7-day PITR</a><br />
<a href="/spanner/docs/backup#backup-schedules">Scheduled backups</a></td>
<td><a href="/spanner/docs/backup">Standard backups</a><br />
<a href="/spanner/docs/pitr">7-day PITR</a><br />
<a href="/spanner/docs/backup#backup-schedules">Scheduled backups</a><br />
<a href="/spanner/docs/backup#incremental-backups">Incremental backups</a></td>
<td><a href="/spanner/docs/backup">Standard backups</a><br />
<a href="/spanner/docs/pitr">7-day PITR</a><br />
<a href="/spanner/docs/backup#backup-schedules">Scheduled backups</a><br />
<a href="/spanner/docs/backup#incremental-backups">Incremental backups</a></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/cuds">CUDs</a></td>
<td>20% for 1 year<br />
40% for 3 years</td>
<td>20% for 1 year<br />
40% for 3 years</td>
<td>20% for 1 year<br />
40% for 3 years</td>
</tr>
</tbody>
</table>

## What you need to do

If you haven't selected an edition for your legacy Spanner instance, it has automatically upgraded to the lowest edition that matches your usage pattern to avoid workload disruption.

In general:

  - Regional instances upgraded to the Standard edition.
  - Regional instances with additional configurable read-only replicas or those using Enterprise edition features upgraded to the Enterprise edition.
  - Multi-region instances or those using Enterprise Plus edition features upgraded to the Enterprise Plus edition.

If you are a Spanner customer under Google Cloud commitments with discounts on legacy SKUs, **no action is required on your part.** You can continue to use legacy SKUs until the expiration of your contract. You also have the option to upgrade to Spanner editions. We recommend contacting your sales team to understand your existing contractual obligation and renew your contracts to include the new editions SKUs, so you can optimize your total cost of ownership and get access to new capabilities that are only offered in editions.

## Monitor edition feature usage

You can monitor the usage of Enterprise edition and Enterprise Plus edition edition features in your instance. To do so, use the [Feature usage](/monitoring/api/metrics_gcp_p_z#gcp-spanner) ( `  instance/edition/feature_usage  ` ) monitoring metric. The following features are shown in this metric when you use them in your instance.

  - Asymmetric autoscaling
  - Columnar engine
  - Full-text search
  - Geo-partitioning
  - Incremental backups
  - KNN vector search: includes use of the KNN vector distance functions
  - Managed autoscaler
  - Scheduled backups
  - Spanner Graph
  - Tiered storage
  - Vector search: includes use of the ANN vector distance functions and vector index

**Note:** The Feature usage metric is sampled every 60 seconds, and might take up to 120 seconds to become visible.

To view the edition feature usage metric in the Google Cloud console, follow these steps:

1.  In the Google Cloud console, go to **Monitoring** :

2.  In the navigation menu, select **Metrics explorer** .

3.  In the **Metric** field, click the **Select a metric** drop-down.

4.  In the **Filter by resource or metric name** field, select **Cloud Spanner Instance \> Instance \> Feature usage** , and then click **Apply** .

5.  In the **Aggregation** field, select **Unaggregated** .

6.  Select **Table** or **Both** as the table type instead of Chart.
    
    The table lists each higher-tier edition feature that is being used by your instance and database.
    
    Optionally, you can click view\_column **Column display options** to display or hide columns to display in the table. The `  name  ` column should generally be ignored in favor of `  instance_id  ` , `  instance_config  ` , `  database  ` and `  feature  ` .

To see a full list of Google Cloud metrics, see [Google Cloud metrics](/monitoring/api/metrics_gcp) .

## Pricing

For information about Spanner editions pricing, see [Spanner pricing](https://cloud.google.com/spanner/pricing) . To help control cost, it is possible to prevent specific Spanner editions from being created by using an [organization policy constraint](/spanner/docs/spanner-custom-constraints) .

## Frequently asked questions

  - **What are the changes if I'm using a Spanner free trial instance?**  
    If you're using a free trial instance, it will default to the Enterprise edition when you [upgrade it to a paid instance](/spanner/docs/free-trial-quickstart#upgrade) . Once in Enterprise edition, you can upgrade the instance to the Enterprise Plus edition, or contact support to downgrade to the Standard edition.

<!-- end list -->

  - **Can I upgrade my instance?**  
    Yes, you can [upgrade your instance](/spanner/docs/create-manage-instances#upgrade-edition) to the Enterprise edition or Enterprise Plus edition. There is no data migration involved when you change the edition of your instance. The edition upgrade takes approximately 10 minutes to complete with zero downtime.
  - **Can I downgrade my instance?**  
    Yes, you can [downgrade your instance](/spanner/docs/create-manage-instances#downgrade-edition) to a lower-tier edition. You must stop using the higher-tier edition features in order to downgrade. There is no data migration involved when you change the edition of your instance. The edition downgrade takes approximately 10 minutes to complete with zero downtime.
  - **Will granular instances continue to be supported with editions?**  
    Yes, granular instances is supported in all Spanner editions. The minimum compute capacity you can set is 100 processing units or one node (1000 processing units).
  - **Will my instance undergo data migration when it upgrades to editions?**  
    No, your instance doesn't undergo data migration when it upgrades to editions. This is a configuration change.

**How do I stop using Enterprise edition or Enterprise Plus edition features in order to downgrade my instance's edition?**

  - Enterprise edition and Enterprise Plus edition features:
    
      - Custom read-only replicas: [Move your instance](/spanner/docs/move-instance) to a regional instance configuration or [delete your instance](/spanner/docs/create-manage-instances#delete-instance) .
      - Spanner Graph: [Delete all property graph schemas](/spanner/docs/graph/create-update-drop-schema#drop-property-graph-schema) in your instance.
      - Full-text search: [Delete all search indexes](/spanner/docs/reference/standard-sql/data-definition-language#drop-search-index) in your instance.
      - Vector search: Stop using all [KNN](/spanner/docs/find-k-nearest-neighbors) and [ANN](/spanner/docs/find-approximate-nearest-neighbors) distance functions, and [delete all vector indexes](/spanner/docs/reference/standard-sql/data-definition-language#drop-vector-index) in your instance.
      - Managed autoscaler: Change your instance from using the managed autoscaler to [use manual scaling](/spanner/docs/create-manage-instances#remove-managed-autoscaler) .

  - Enterprise Plus edition only features:
    
      - Dual-region and multi-region instance configurations: [Move your instance](/spanner/docs/move-instance) to a regional instance configuration or [delete your instance](/spanner/docs/create-manage-instances#delete-instance) .
      - Geo-partitioning: [Delete all partitions](/spanner/docs/create-manage-partitions#delete-partition) in your instance.

## What's next

  - Learn how to [create and manage your instances](/spanner/docs/create-manage-instances) .
