**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This page describes and explains how tiered storage works in Spanner. This feature is supported in both GoogleSQL-dialect and PostgreSQL-dialect databases.

Spanner tiered storage is a fully-managed storage feature that lets you choose whether to store your data on solid-state drives (SSD) or hard disk drives (HDD). By default, when you're not using tiered storage, your data is stored on SSD storage. Depending on how often you use or access the data, you might consider using tiered storage and storing data on both SSD and HDD storage.

  - **SSD storage** is the most performant (higher queries per second) and cost-effective choice for most use cases. You should use it to store active data with high write and read throughput and data that requires low-latency data access.
  - **HDD storage** is sometimes appropriate for large datasets that aren't latency-sensitive, are infrequently accessed, or if the cost of storage is an important consideration.

Using tiered storage lets you take advantage of both SSD storage, which supports the high performance of active data, and HDD storage, which supports infrequent data access at a lower cost.

## Choose between SSD and HDD storage

The following table lists the differences and similarities between SSD and HDD storage. The throughput figures in the table are per node and scale linearly with the number of nodes in your Spanner instance. When in doubt, we recommend that you choose SSD storage.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;"></th>
<th style="text-align: left;">SSD storage</th>
<th style="text-align: left;">HDD storage</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;"><strong>Target use cases</strong></td>
<td style="text-align: left;">Data that requires high write and read throughput, and low-latency data access</td>
<td style="text-align: left;">Large datasets that aren't latency-sensitive or are infrequently accessed</td>
</tr>
<tr class="even">
<td style="text-align: left;"><strong>Expected throughput per node</strong><br />
<strong>Regional configurations</strong></td>
<td style="text-align: left;">Up to 22,500 QPS read<br />
Up to 3,500 QPS write</td>
<td style="text-align: left;">Up to 1,500 QPS read<br />
Up to 3,500 QPS write</td>
</tr>
<tr class="odd">
<td style="text-align: left;"><strong>Expected throughput per node</strong><br />
<strong>Dual-region and multi-region configurations</strong></td>
<td style="text-align: left;">Up to 15,000 QPS read per region<br />
Up to 2,700 QPS write</td>
<td style="text-align: left;">Up to 1,000 QPS read per region<br />
Up to 2,700 QPS write</td>
</tr>
<tr class="even">
<td style="text-align: left;"><strong>Supported operations</strong></td>
<td style="text-align: left;">Read, write, update, and delete</td>
<td style="text-align: left;">Read, write, update, and delete</td>
</tr>
</tbody>
</table>

Use [throughput optimized writes](/spanner/docs/throughput-optimized-writes) to increase write throughput beyond the numbers in the table. For more information, see [Performance overview](/spanner/docs/performance) .

## Benefits

Tiered storage offers the following benefits by letting you use both SSD and HDD storage:

  - **Significant total cost of ownership reduction** : HDD storage provides a lower cost option for large datasets that aren't latency-sensitive or are infrequently accessed.
  - **Ease of management** : Provides a fully-managed tiering service without the complexity of additional pipelines and split logic.
  - **Unified and consistent experience** : Provides unified data access and a single set of metrics across hot and (mutable) cold data
  - **Enhanced performance** : Improves query performance by organizing your data in different locality group, which provides data locality and isolation across columns. Data in the same locality group is stored physically close together.

## How tiered storage works

By default, when you create a new instance, data is only stored on SSD storage. Similarly, data in existing instances is also only stored on SSD storage.

If you choose to use tiered storage to store some data in HDD storage, you must create a [locality group](/spanner/docs/schema-and-data-model#locality-groups) , which is used to define the tiered storage policy for data in your schema. When you create a locality group, you can define the storage type, either `  ssd  ` or `  hdd  ` . Optionally, you can also define the amount of time that data is stored on SSD storage before it's moved to HDD storage. This time is relative to the data's commit timestamp. After the specified time passes, Spanner migrates the data to HDD storage during its normal compaction cycle, which typically occurs over the course of seven days from the specified time. This is known as an age-based tiered storage policy. When using an age-based tiered storage policy, the minimum amount of time that data must be stored on SSD before it's moved to HDD storage is one hour.

With your locality groups defined, when you create your tables, you can set the tiered storage policy at the database, table, column, or secondary index-level. The tiered storage policy determines how and where data is stored. For instructions, see [Create and manage locality groups](/spanner/docs/create-manage-locality-groups) .

### Key behavior details

The following behaviors apply when using tiered storage:

  - **Storage tiers are applied at the cell level** : Age-based tiered storage policies apply to individual cells within a row, based on the cell's commit timestamp. Data is moved to HDD storage if its commit timestamp is older than the specified duration. A single row can therefore have some data on SSD and some on HDD, depending on when each cell was last updated.
  - **Interleaved tables are independent** : Locality group settings and tiered storage policies aren't inherited by interleaved child tables. You must apply these settings to each table individually.
  - **Read operations don't affect data aging** : Reading data from a table doesn't reset its age or prevent it from being moved to HDD. Only a write operation that updates the data resets its commit timestamp and keeps it in SSD storage.
  - **Moving data from HDD back to SSD** : When you update data stored on HDD, Spanner writes the new version of the data directly to SSD storage. The previous version of the data on HDD is marked as obsolete and is cleaned up later by a background compaction process. This ensures that updates to cold data immediately benefit from SSD performance for subsequent reads of the new data.

## Back up and restore

You can back up and restore your data using [Spanner backups](/spanner/docs/backup) . The backup contains all storage schema information, including [`  INFORMATION_SCHEMA.LOCALITY_GROUP_OPTIONS  `](#information-schema) , which specifies the storage type of each locality group. To restore a backup that contains locality groups to a new instance, the destination instance must be in the Spanner Enterprise edition or Spanner Enterprise Plus edition.

## Data Boost

You can use [Spanner Data Boost](/spanner/docs/databoost/databoost-overview) to access data on SSD or HDD storage. Querying data on HDD uses the instance's [HDD disk load capacity](/monitoring/api/metrics_gcp_p_z#spanner/instance/disk_load) , which is part of your compute capacity.

## Search indexes

[Full-text search](/spanner/docs/full-text-search) and [vector indexes](/spanner/docs/find-approximate-nearest-neighbors#vector-index) inherit the locality group that is set on the [database object](/spanner/docs/create-manage-locality-groups#set_a_database-level_locality_group) .

## Observability

The following observability features are available for tiered storage.

### Cloud Monitoring metrics

Spanner provides the following metrics to help you monitor your tiered storage usage and data using Cloud Monitoring:

  - `  spanner.googleapis.com/instance/storage/used_bytes  ` (Total storage): Shows the total bytes of data stored on SSD and HDD storage.
  - `  spanner.googleapis.com/instance/storage/combined/limit_bytes  ` : Shows the combined SSD and HDD storage limit.
  - `  spanner.googleapis.com/instance/storage/combined/limit_bytes_per_processing_unit  ` : Shows the combined SSD and HDD storage limit for each processing unit.
  - `  spanner.googleapis.com/instance/storage/combined/utilization  ` : Shows the combined SSD and HDD storage utilization, compared against the combined storage limit.
  - `  spanner.googleapis.com/instance/disk_load  ` : Shows the HDD usage in percentage. If your instance reaches 100% HDD disk load, you experience significant increased latency.

If you have existing queries that filter existing metrics by `  storage_class:ssd  ` , you must remove the filter to see your HDD usage.

To learn more about monitoring your Spanner resources, see [Monitor instances with system insights](/spanner/docs/monitoring-console) and [Monitor instances with Cloud Monitoring](/spanner/docs/monitoring-cloud) .

### Information schema

`  INFORMATION_SCHEMA.LOCALITY_GROUP_OPTIONS  ` contains the list of locality groups and options in your Spanner database. It includes information for the `  default  ` locality group. For more information, see [`  locality_group_options  ` for GoogleSQL-dialect databases](/spanner/docs/information-schema) and [`  locality_group_options  ` for PostgreSQL-dialect databases](/spanner/docs/information-schema-pg) .

### Built-in statistics tables

The following built-in statistics tables are available for databases using tiered storage:

  - `  SPANNER_SYS.TABLE_SIZES_STATS_1HOUR  ` : Shows HDD and SSD storage usage for each table in your database.
  - `  SPANNER_SYS.TABLE_SIZES_STATS_PER_LOCALITY_GROUP_1HOUR  ` : Shows HDD and SSD storage usage for each locality group in your database.

For more information, see [Table sizes statistics](/spanner/docs/introspection/table-sizes-statistics) .

The query statistics and read statistics tables have the following column that is related to tiered storage:

  - `  AVG_DISK_IO_COST  ` : The average cost of this query in terms of Spanner [HDD disk load](/monitoring/api/metrics_gcp_p_z#spanner/instance/disk_load) . Use this value to make relative HDD I/O cost comparisons between reads that you run in the database. A higher value indicates that you are using more HDD disk load and your query might be slower than if it was running on SSD. Furthermore, if your HDD disk load is at capacity, the performance of your queries might be further impacted.

For more information, see [Query statistics](/spanner/docs/introspection/query-statistics) and [Read statistics](/spanner/docs/introspection/read-statistics) .

## Pricing

There is no additional charge for using tiered storage. You are charged the standard Spanner pricing for the amount of compute capacity that your instance uses and the amount of storage that your database uses. Data that is stored on SSD and HDD is billed at their respective storage rates. You aren't charged for moving data between SSD and HDD storage. Querying data on HDD uses the instance's [HDD disk load capacity](/monitoring/api/metrics_gcp_p_z#spanner/instance/disk_load) , which is part of your compute capacity pricing. For more information, see [Spanner pricing](https://cloud.google.com/spanner/pricing) .

## What's next

  - Learn more about [locality groups](/spanner/docs/schema-and-data-model#locality-groups) .
  - Learn how to [create and manage locality groups](/spanner/docs/create-manage-locality-groups) .
  - Learn more about [optimizing queries with timestamp predicate pushdown](/spanner/docs/sql-best-practices#optimize-timestamp-predicate-pushdown) .
