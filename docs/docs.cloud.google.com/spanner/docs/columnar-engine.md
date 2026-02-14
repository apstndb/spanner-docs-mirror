**Preview — [Spanner columnar engine](/spanner/docs/columnar-engine)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This page provides an overview of the columnar engine for Spanner and describes how to use it.

Operational databases commonly extract, transform, and load (ETL) data into an OLAP system for analytics. This system is often part of a data warehouse. With [Data Boost for Spanner](/spanner/docs/databoost/databoost-overview) , Spanner already separates analytical compute, which ensures transactional stability.

Columnar engine is a storage technique that analytics systems use to speed up scans when compared to batch-based scans. Spanner columnar engine lets you run analytics with significantly improved performance on the latest operational data. Spanner columnar engine increases scan performance by up to 200 times, eliminating the need for ETL while maintaining strong consistency.

Spanner's [Ressi](https://cloud.google.com/blog/products/databases/spanner-modern-columnar-storage-engine) format uses a partition attributes across (PAX) column-wise layout for efficient scans within a data block. However, this format colocates all columns of a row within a given block for fast single-row lookups. Unlike Ressi, Spanner's columnar engine dedicates runs of blocks to a single column. This approach is more efficient for sequential scans, as Spanner only needs to read the columns referenced in the query.

Spanner builds the columnar representation in the background (as part of compactions), and automatically merges the representation with the latest updates at query time to provide strong consistency. Queries that wouldn't benefit from columnar storage can continue to use PAX.

Workloads that would benefit from using columnar engine include the following:

  - Operational reporting extract up-to-the-second business intelligence from the latest operational data.
  - Served analytics power dashboards and custom drill-downs with interactive latency.
  - Federated analytics seamlessly combine data from Spanner and other sources in BigQuery.

[Spanner instance backups](/spanner/docs/backup) don't include the columnar format.

## Best practices for using columnar engine

This section describes best practices when using columnar engine.

### Large scan optimization

Columnar engine optimizes queries that scan large amounts of data. For smaller data scans or queries with quickly satisfied `  LIMIT  ` clauses, row-based scans might be more efficient.

### Essential columns

If you use `  SELECT *  ` , Spanner reads all columns from columnar storage. To maximize performance, specify only necessary columns. For example, `  SELECT column1, column2 FROM ...  ` .

### Performance bottleneck identification

Columnar engine is effective for scan-bound workloads. To identify a scan-bound workload, check the [query plan](/spanner/docs/query-execution-plans) for a high latency level in the **Table scan** node. If your query isn't scan-bound, prioritize other optimizations first. Columnar engine can provide benefits later if your optimizations make the query scan-bound.

### Optimal columnar coverage

After you [enable columnar engine](/spanner/docs/configure-columnar-engine#enable-columnar-engine) on a database that already contains data, Spanner's automatic compaction process converts data to columnar storage asynchronously in the background. To see how much your query benefits, check the [**Columnar read share** percentage](/spanner/docs/monitor-columnar-engine#columnar-data-query-plan) in the query plan.

### High churn data management

High write rates from updates or random inserts can affect the columnar engine's performance. Append-only workloads experience minimal impact from using columnar engine. Compaction is a background process, which typically is spread out over multiple days, but can happen sooner if the size of the database grows substantially. Alternatively, design the schema to favor append-only writes at the split level. For more information, see [sharding of timestamp-ordered data in Spanner](https://cloud.google.com/blog/products/gcp/sharding-of-timestamp-ordered-data-in-cloud-spanner) .

### Workload isolation

There are two techniques that you can use to isolate analytical queries from transactions:

  - Use [directed reads](/spanner/docs/directed-reads) to route reads to read-only replicas.
  - Use [Data Boost for federated queries](/spanner/docs/databoost/databoost-run-queries) .

## Pricing

Billing for the Spanner columnar engine is based on storage usage. After you enable the Spanner columnar engine and Spanner completes data compaction, storage usage increases to include the new columnar representation. Columnar engine provides storage metrics that let you monitor the impact to storage. For more information, see [Columnar data storage metrics](/spanner/docs/monitor-columnar-engine#monitor-storage-usage) .

Spanner columnar engine isn't impacted by the [8 bytes per cell overhead](/spanner/docs/reference/standard-sql/data-types#storage_size_for_data_types) .

## Preview limitations

  - Columnar engine only supports the GoogleSQL interface.

## What's next

  - Learn how to [enable columnar engine](/spanner/docs/configure-columnar-engine) .
  - Learn how to [query columnar data](/spanner/docs/query-columnar-data) .
  - Learn how to [monitor columnar engine](/spanner/docs/monitor-columnar-engine) .
