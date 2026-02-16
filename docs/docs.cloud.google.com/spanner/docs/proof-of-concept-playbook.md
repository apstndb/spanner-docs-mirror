This page provides a strategy for planning and running a proof of concept (POC) with Spanner. It provides in-depth references and insights into crucial aspects of a POC, such as instance configuration, schema design, data loading, and performance evaluation. It highlights essential steps for evaluating Spanner's capabilities and helps you identify potential risks and benefits tied to Spanner adoption.

In addition to validating Spanner's technical capabilities, a POC serves two purposes:

  - To help you understand the benefits that Spanner offers for your use case
  - To help you identify the risks associated with adopting Spanner

A Spanner POC encompasses a variety of evaluation facets, each customized to address your specific business and technical objectives, as shown in the following diagram.

The guidelines in this document help you evaluate each of these areas.

  - **Performance and scalability** helps you understand how Spanner handles specific workloads, latency requirements, and the impact of various instance configurations. These tests can demonstrate Spanner's ability to seamlessly scale.

  - **Monitoring capabilities** helps you assess whether Spanner provides the insights needed for effective database operations. This evaluation includes:
    
      - Options for analyzing query execution plans
      - System resource utilization
      - Options for configuring alerts
    
    A POC can reveal gaps that need to be addressed to fully optimize operational efficiency.

  - **Security and compliance** are key to determining Spanner's suitability for your organization. This includes assessments to ensure Spanner can mitigate security risks while delivering robust compliance benefits, such as the following:
    
      - Encryption options, such as [CMEK or EKM](/spanner/docs/cmek) for data that's in-transit and at-rest
      - Least-privilege access control posture
      - Audit logging
      - Adherence to regulatory requirements

  - **Backup and disaster recovery (DR) capabilities** are essential to ensure operational and data resilience. A POC can validate Spanner's DR features, such as [point-in-time recovery](/spanner/docs/pitr) and availability.

  - **Migration feasibility** involves understanding the complexity of transitioning from your current database solution to Spanner. Evaluating schema compatibility, migration tools, and application changes help you quantify required investments, and determine the risks and benefits of Spanner adoption.

During your evaluation, you might want to explore Spanner's feature set to ensure it meets your application's functional requirements. This might include testing its global consistency, SQL query capabilities, or integration with other Google Cloud services.

While evaluations can highlight Spanner's unique strengths, such as consistency across regions, they can also surface potential risks, such as integration efforts with your existing application architecture.

## POC activities lifecycle

This POC walks you through the following steps. Follow the recommendations in this document to set up and assess Spanner for your specific use case.

## Plan your POC

The foundation of a successful POC lies in defining clear, measurable goals that align with both technical and business priorities. Avoid vague objectives like *explore Spanner's potential* , as they often lead to unfocused efforts and ambiguous outcomes. Instead, tie your POC goals to concrete targets, such as achieving 99.999% availability, reducing downtime, or scaling to handle a 200% increase in throughput while maintaining transaction latencies under 20ms.

Spanner's unique architecture is ideal for workloads requiring massive scalability, so assessing scalability for your use case is a good starting point. Test scenarios should include:

  - Handling typical operational loads
  - Managing traffic surges
  - Scaling back down efficiently

These tests help you understand how Spanner performs under different conditions and whether it meets your technical requirements for scalability. Specific, actionable goals not only help structure the POC but also create a solid basis for evaluating success.

### Define a quantified evaluation rubric

A rubric consisting of clear, measurable metrics and discrete success criteria is essential to conclude whether the POC met its objectives. For example, rather than only testing performance, you should also specify goals such as:

  - Serve specific production-level QPS (queries per second)
  - Maintain sub-20ms latencies under predefined peak loads
  - Handle clearly-defined traffic bursts with no performance degradation

Well-defined criteria help you evaluate Spanner objectively for your workload and provide actionable insights for next steps. Be specific and define percentile targets for read and write operation latency (such as p50 and p95). A clear definition of acceptable latency thresholds help you design tests of Spanner performance that align with your business needs.

An example for an evaluation rubric might look something like the following:

<table>
<tbody>
<tr class="odd">
<td><strong>Evaluation Facet</strong></td>
<td><strong>Success Criteria</strong></td>
</tr>
<tr class="even">
<td>Availability</td>
<td>99.999%</td>
</tr>
<tr class="odd">
<td>Security</td>
<td>CMEK with an EKM required</td>
</tr>
<tr class="even">
<td>Recovery Point Objective (RPO) guarantee in case of a regional outage</td>
<td>0</td>
</tr>
<tr class="odd">
<td>Latency limit for most critical transactions</td>
<td>p50 below 20ms</td>
</tr>
<tr class="even">
<td>Latency for our most critical user-facing queries</td>
<td>p50 below 100ms</td>
</tr>
<tr class="odd">
<td>Scalability</td>
<td>Demonstrate that scaling up from 10,000 transactions per second to 100,000 transactions per second with p50 latency below 20ms is possible over the course of one hour</td>
</tr>
</tbody>
</table>

### Scope your evaluation cases

A POC shouldn't require a full-scale migration. Instead, focus on testing representative workloads or critical components of your system. For example, identify key queries, critical transaction shapes, or specific data-driven workflows that are central to your operations. Narrow the scope to reduce complexity while ensuring that the results are relevant and meaningful. This approach provides a manageable way to evaluate Spanner's capabilities without being overwhelmed by the intricacies of an entire system migration.

## Choose a Spanner instance configuration

When you create a Spanner instance for evaluation purposes, choose an [instance configuration](/spanner/docs/instance-configurations) that meets your business requirements for geographic location and service availability SLA. Spanner offers various configurations, including single-region, multi-region, and dual-region. Each configuration is designed to meet different latency, availability, and redundancy requirements.

  - **Single-region configurations** store data in one Google Cloud region, offering low latency within that region and cost effectiveness. These topologies are ideal for workloads that require intra-region zonal redundancy that provides availability of 99.99%.
  - **Dual-region configurations** replicate data across two regions in a single country with a witness replica in each region for failovers. This configuration provides higher availability (99.999%) and fault tolerance than a single-region setup. These topologies are well-suited for workloads with tight compliance (such as [data residency](/spanner/docs/data-residency) ) or geographic proximity requirements.
  - **Multi-region configurations** replicate data across multiple regions, ensuring a very high availability and resilience to regional outages. These topologies are ideal for applications requiring geo-redundancy with an availability up to 99.999%.

### Latency considerations in cross-region instances

In dual- and multi-region configurations, the geographic distribution of Spanner replicas can influence latency. Write latency depends on the proximity of the leader region, which coordinates read-write transactions, and the other regions, which confirm each write operation. Placing your application's compute resources close to the leader region reduces round-trip delays and minimizes latency.

You can [modify the leader region of a database](/spanner/docs/modifying-leader-region) to align with your application's needs. For read-only operations, Spanner can serve [stale reads](/spanner/docs/reads) from the closest replica, reducing latency, while strong reads might involve the leader region, potentially increasing operation latency. To optimize latency in multi-region setups, choose the leader region strategically, colocate the compute resources for your services with the leader region, and take advantage of stale reads for read-heavy workloads.

### Configurations that meet your application's requirements

When you select an instance configuration for your application, consider factors such as availability, latency, and data residency requirements. For example, if your application requires low-latency responses for users in a specific geographic area, a regional instance might be sufficient. However, for applications that demand higher availability or serve globally distributed users, multi-region configurations would be more appropriate.

Start with a configuration that closely aligns with your application's production requirements to evaluate performance. Keep in mind that latency and costs vary between configurations, so tailor your POC environment to reflect the needs of your use case. For multi-region deployments, simulate geographic service distribution and test latency to ensure the configuration aligns with your production requirements. For more details, see [Spanner multi-region leader placement guidance](https://cloud.google.com/blog/topics/developers-practitioners/reduce-latency-with-cloud-spanner-multi-region-leader-placement) .

## Spanner sizing

Provision the initial compute capacity for your Spanner instance to ensure it can handle your evaluation workload effectively during the POC. The initial instance size should align with the expected workload, factoring in the mix of read and write queries per second (QPS), query complexity, and concurrency levels.

Starting with a reasonable assumption lets you establish a baseline and incrementally scale based on observed performance. You can use the sizing guidance from Spanner's [reference benchmarks](/spanner/docs/performance) to establish a baseline instance configuration.

Sizing during a POC should be iterative. Begin with an initial setup, then monitor key metrics like latency and CPU utilization, and adjust assigned compute capacity as needed. This ensures that you can validate Spanner's scalability and performance capabilities while replicating conditions similar to your production environment.

Typical workload patterns, such as consistent traffic versus fluctuating demand, should influence your sizing approach. When you enable [autoscaling](/spanner/docs/autoscaling-overview) , Spanner provisions its compute resource capacity dynamically to match workload intensity.

## Schema design

Schema design is a critical aspect of a Spanner POC because the way that you organize your data can directly impact performance and scalability.

A well-designed schema is foundational for demonstrating Spanner's capabilities in a POC. Load tests often reveal potential bottlenecks or inefficiencies, informing iterative refinements that create an optimal schema.

### Design for scalability

When you create a database schema for Spanner, it's essential to take its distributed architecture into account. Some key considerations and optimizations include the following:

  - **Primary keys:** choose primary keys that distribute data evenly over the key space, avoiding monotonically increasing keys like timestamps that might cause hotspots on splits.
  - **Indexes:** design indexes to optimize query performance while being mindful of their impact on write performance and storage costs. Too many or poorly planned indexes might introduce unnecessary overhead.
  - **Table interleaving:** use table interleaving to optimize access patterns for related data. This might reduce cross-process communication and improve query efficiency.

See the [Spanner schema design best practices](/spanner/docs/schema-design) to avoid common pitfalls and design a schema that supports high performance and scalability.

You can create a draft schema in the Google Cloud console as shown in the following image.

### Schema migration with the Spanner migration tool

The [Spanner migration tool (SMT)](https://googlecloudplatform.github.io/spanner-migration-tool/) can simplify schema creation when migrating from relational databases, including MySQL or PostgreSQL. SMT automates schema generation and includes basic optimizations, such as suggesting indexes and schema adjustments. While SMT provides a strong starting point, manual refinements are often needed to align the schema with your specific use cases or workload patterns.

### Use an iterative schema design process

While an initial schema provides a starting point, it's unlikely to be perfect. Schema creation for a POC is not a one-time task but an iterative process that evolves as you gain insights from your testing. A robust schema is essential for application performance; achieving this involves a well-thought-out initial design, leveraging tools like SMT, and iterative refinement based on load testing outcomes. By following this process, you can ensure your schema effectively meets your application's demands. You also learn how to best take advantage of Spanner features.

## Data loading

A successful Spanner POC relies on loading representative data into the database to validate schema design and simulate application workflows. There are several recommended tools that can streamline this process. To load your own data, Spanner provides the following options:

  - [BigQuery's reverse extract, transform, and load (ETL)](#bq-reverse-etl) to Spanner is an easy-to-use, integrated data loading mechanism that lets you use SQL-based transformations to load data into Spanner. This method is ideal for a wide range of data formats, including semi-structured data like JSON.
  - For relational databases like MySQL and PostgreSQL, the [Spanner migration tool (SMT)](#smt) automates schema creation, data type mapping, and bulk data loading.
  - For flat-file formats, Google provides [Dataflow templates](#dataflow-templates) for CSV to Spanner and Avro to Spanner to create manual schema definitions for bulk data loading. For JDBC-compatible databases, Google provides the JDBC to Spanner Dataflow template.

For more information about these options, see [Bring your own data](#bring-your-own-data) .

If no sample data is available, you can use synthetic data generation tools such as [JMeter from Machmeter](#jmeter) and [QuickPerf](#quickperf) to help you create datasets tailored to your schema and use case. For more information, see [Generate sample data](#generate-sample-data) .

### Bring your own data

If you have available sample data that you want to use for the POC, you have several options for loading that data into Spanner.

<table>
<tbody>
<tr class="odd">
<td><strong>Source</strong></td>
<td><strong>Tool</strong></td>
<td><strong>Schema creation</strong></td>
<td><strong>Transformations</strong></td>
<td><strong>Data Size</strong></td>
</tr>
<tr class="even">
<td><strong>MySQL</strong></td>
<td><a href="https://googlecloudplatform.github.io/spanner-migration-tool/">SMT</a></td>
<td>automatic</td>
<td>Data type conversion only</td>
<td>small</td>
</tr>
<tr class="odd">
<td><strong>PostgreSQL</strong></td>
<td><a href="https://googlecloudplatform.github.io/spanner-migration-tool/">SMT</a></td>
<td>automatic</td>
<td>Data type conversion only</td>
<td>small</td>
</tr>
<tr class="even">
<td><strong>Any JDBC</strong></td>
<td><a href="/dataflow/docs/guides/templates/provided/sourcedb-to-spanner">JDBC to Spanner</a></td>
<td>manual</td>
<td>Data type conversion only</td>
<td>large</td>
</tr>
<tr class="odd">
<td><strong>CSV</strong></td>
<td><a href="/dataflow/docs/guides/templates/provided/cloud-storage-to-cloud-spanner">CSV to Spanner</a></td>
<td>manual</td>
<td>Data type conversion only</td>
<td>large</td>
</tr>
<tr class="even">
<td></td>
<td><a href="/bigquery/docs/export-to-spanner">BigQuery reverse ETL</a></td>
<td>manual</td>
<td>Complex transformations supported</td>
<td>large</td>
</tr>
<tr class="odd">
<td><strong>Avro</strong></td>
<td><a href="/dataflow/docs/guides/templates/provided/avro-to-cloud-spanner">Avro to Spanner</a></td>
<td>manual</td>
<td>Data type conversion only</td>
<td>large</td>
</tr>
<tr class="even">
<td></td>
<td><a href="/bigquery/docs/export-to-spanner">BigQuery reverse ETL</a></td>
<td>manual</td>
<td>Complex transformations supported</td>
<td>large</td>
</tr>
<tr class="odd">
<td><strong>JSON</strong></td>
<td><a href="/bigquery/docs/export-to-spanner">BigQuery reverse ETL</a></td>
<td>manual</td>
<td>Complex transformations supported</td>
<td>large</td>
</tr>
</tbody>
</table>

#### BigQuery reverse ETL to Spanner

[BigQuery reverse ETL to Spanner](/bigquery/docs/export-to-spanner) lets you [quickly ingest a wide range of data sources](https://cloud.google.com/blog/topics/developers-practitioners/bigquery-explained-data-ingestion) and transform them into BigQuery tables using SQL. You can then [export data from the BigQuery table into a Spanner table](/bigquery/docs/export-to-spanner#export_data) . It's particularly useful for semi-structured data, such as JSON, which often originates as exports from NoSQL data sources. While BigQuery has automatic schema detection in place, Spanner schema creation is manual, requiring that you define the schema before loading data.

#### Spanner migration tool

To jumpstart your POC, you can use the [Spanner migration tool (SMT)](https://googlecloudplatform.github.io/spanner-migration-tool/) to migrate data from MySQL and PostgreSQL sources into Spanner. SMT automates the schema creation process, mapping data types from the source database to their equivalent types in Spanner. It also makes Spanner-specific schema optimization recommendations. This makes it particularly useful for straightforward migrations where automated schema conversion is sufficient.

The SMT provides a user interface that guides you through the migration process. During this process, you select the source database, and review recommendations and options for schema design.

#### Dataflow templates

[Dataflow](/dataflow/docs/overview) is a fully managed service designed for scalable data processing, making it a suitable choice for loading large quantities of data.

Google provides the following open source templates for common loading patterns:

  - [CSV to Spanner](/dataflow/docs/guides/templates/provided/cloud-storage-to-cloud-spanner) loads data from CSV files stored in Cloud Storage into Spanner.
  - [Avro to Spanner](/dataflow/docs/guides/templates/provided/avro-to-cloud-spanner) loads existing Avro data files from Cloud Storage.
  - [JDBC to Spanner](/dataflow/docs/guides/templates/provided/sourcedb-to-spanner) loads data from databases that support JDBC.

Each of these templates requires that you manually create the Spanner schema before starting the data load.

Dataflow automatically scales out to accommodate datasets of any size, ensuring high-performance data ingestion into Spanner, even for terabyte-scale datasets. This scalability is provided at the expense of a few tradeoffs:

  - Dataflow pipelines require manual configuration to define the schema, data mapping, and execution parameters for optimal execution.
  - Dataflow provides the flexibility and power needed for large-scale data migrations, but it might require more effort to set up and manage than other tools.

### Generate sample data

If you don't have sample data but have a specific use case in mind, you can model the schema based on your requirements, and use tools to generate representative datasets. These tools let you populate Spanner with meaningful data to validate your schema design and application workflows.

#### JMeter from Machmeter

[JMeter from Machmeter](https://cloudspannerecosystem.dev/machmeter/) provides examples that use [JMeter](https://jmeter.apache.org/) to generate sample data for Spanner. Machmeter's focus on use-case-driven examples makes it a great starting point for generating data patterns structurally similar to your expected production schema. The provided examples include scripts for bulk inserts and other operations. You can adapt the scripts to generate synthetic datasets at scale. For more information, see the Machmeter [repository](https://github.com/cloudspannerecosystem/machmeter/tree/master/machmeter/usecases) or [documentation](https://cloudspannerecosystem.dev/machmeter/) .

#### QuickPerf

[QuickPerf](https://github.com/googleapis/java-spanner-jdbc/tree/main/samples/quickperf) is distributed with the Spanner JDBC driver. QuickPerf provides SQL-based scripts that quickly create representative datasets for testing schema integrity and database behavior. This is a low-effort choice for quickly generating small to medium-sized datasets that are less complex.

## Load testing

Load testing lets you observe Spanner performance when handling workloads to ensure your database has the optimal configuration for production demands. Two tools introduced previously, [JMeter from Machmeter](https://cloudspannerecosystem.dev/machmeter/) and [QuickPerf](https://github.com/googleapis/java-spanner-jdbc/tree/main/samples/quickperf) , are particularly effective for simulating workloads and measuring performance metrics such as throughput, latency, and resource utilization.

[Apache JMeter](https://jmeter.apache.org/) , enhanced through the Machmeter project, provides a powerful framework for distributed load testing with Spanner. Machmeter includes prebuilt JMeter configurations specifically designed for simulating Spanner workloads. These configurations can be tailored to execute representative queries, transactions, and batch operations, letting you measure Spanner's performance under different scenarios.

JMeter's ability to simulate concurrent users and transactions makes it a good choice for testing the scalability and resilience of your Spanner instance. You can deploy JMeter in a distributed mode using Kubernetes or the managed service [GKE](/kubernetes-engine/docs) to scale your test environment. The results offer insights into how Spanner manages specific workloads, scales under increasing demand, and performs during peak loads.

For more information and example configurations, see the [Machmeter repository](https://github.com/cloudspannerecosystem/machmeter/tree/master/machmeter/usecases) .

[QuickPerf](https://github.com/googleapis/java-spanner-jdbc/tree/main/samples/quickperf) is a lightweight benchmarking tool designed for performance testing with Spanner. It focuses on generating performance metrics with minimal setup, letting you quickly iterate on optimizations. QuickPerf is easy to configure and is particularly well-suited for smaller-scale tests and scenarios where you want to quickly measure the performance impact of specific schema or query optimizations.

### Best practices for load testing

When conducting load tests, it's critical to follow Spanner's best practices to ensure accurate and actionable results.

  - **Warm-up period:** allow a warm-up period (typically 30 minutes or more) for Spanner to reach a steady state after scaling nodes or introducing a new workload.
  - **Measure relevant metrics:** focus on metrics, such as throughput (operations per second), latencies percentiles (for example, p50, p95), and CPU utilization to understand how Spanner serves your workload.
  - **Run long benchmarks:** for more representative results, run your load tests for extended periods (for example, over one hour) to account for system behaviors like rebalancing and background maintenance tasks.
  - **Scaling tests:** test both scale-up and scale-down scenarios to observe Spanner behavior under different node configurations and peak loads.

You can use tools like JMeter Machmeter and QuickPerf, along with best practices for load testing, to effectively evaluate Spanner's performance, identify bottlenecks, and optimize your database to meet the demands of your workload.

## Monitoring

To effectively demonstrate the performance and scalability of Spanner during a POC, especially under load, requires that you have a deep understanding of its operational characteristics. Spanner provides a comprehensive suite of monitoring and diagnostic tools designed to give you granular insights into every aspect of your database's performance. This toolset offers a range of resources, from metric dashboards to detailed system tables, that help you identify bottlenecks, validate design choices, and optimize performance.

**[System insights](/spanner/docs/monitoring-console)** provides in-depth observability into the performance and operational health of a Spanner instance. It offers metrics and insights into several areas, including CPU utilization, latency, throughput, and more, at adjustable levels of granularity. During a POC, this is the starting point for observing Spanner behavior during your tests. System insights lets you quickly identify performance bottlenecks, such as high CPU utilization or increased read or write latencies. It sets the foundation for subsequent investigations.

**[Query insights](/spanner/docs/using-query-insights)** provides a top-down view of query execution, starting with identifying the most frequent and costly queries based on metrics like CPU time, execution count, and average latency. Going deeper, query insights lets you examine detailed [execution plans](/spanner/docs/query-execution-plans) , including statistics for each step of the query, and pinpoints specific operations that cause slowdowns. It also offers features that investigate historical performance trends and compare query performance across different time periods. This helps you identify regressions or the impact of schema and code changes. Additional tools, such as the [index advisor](/spanner/docs/index-advisor) , analyzes your queries to recommend new or altered indexes that can improve your query performance.

**[Transaction insights](/spanner/docs/use-lock-and-transaction-insights#txn-insights)** provide visibility into transaction performance with detailed metrics on transaction latency, commit wait times, the number of rows and bytes read and written, and participants in distributed transactions. These metrics reveal high latency or aborted transactions and provide details about their characteristics. During a POC load test, transaction insights is essential for assessing the transactional efficiency of the system under stress. It lets you monitor and identify any degradation as load increases. Analyzing individual transactions helps you pinpoint the causes of slowdowns, such as long-running transactions blocking others or single transactions reading or writing excessively large data volumes. The information from transaction insights lets you perform targeted tuning, like optimizing transaction boundaries, refining queries within transactions, or adjusting the schema to reduce the amount of data involved in typical transactions. This ensures the POC demonstrates Spanner's ability to maintain transactional consistency and performance at the expected load level.

**[Lock insights](/spanner/docs/use-lock-and-transaction-insights#lock-insights)** provides visibility into transaction locking behavior, helping you identify and resolve lock contention issues. It surfaces information about lock waits, including the specific row key ranges that are causing the problem. During a POC load test, lock insights is crucial for identifying whether transactional lock conflicts are causing scalability limitations. As concurrent load increases, transactions might start competing to update the same data, leading to increased wait times and reduced throughput. This information helps you with schema optimization, transaction boundary modification, and application logic adjustments. These actions mitigate contention and ensure that the Spanner database maintains performance under the projected workload, preventing degradation due to locking mechanisms.

**[Hotspot insights](/spanner/docs/find-hotspots-in-database)** identifies performance bottlenecks, specifically increased latency, that result from hotspotting conditions. Hotspots typically occur when there is a high and uneven load. Often, the cause for hotspots are:

  - Suboptimal schema design
  - Primary key selection
  - Access patterns that concentrate operations on a small subset of data rather than distributing them evenly across nodes

During a POC load test, hotspot insights help you decide where to optimize your schema. For example, you might need to [adjust the primary keys](/spanner/docs/schema-design#primary-key-prevent-hotspots) or modify the [secondary indexes](/spanner/docs/secondary-indexes) to prevent hotspots.

**[Key Visualizer](/spanner/docs/key-visualizer)** provides a visual representation of database usage patterns over time across the entire key space of both tables and indexes. It generates heatmaps that show read and write activity, highlighting areas of high intensity and potentially [problematic patterns](/spanner/docs/key-visualizer/patterns) . During a POC, this tool helps to validate schema design and identify potential scalability limitations. As the load increases, you can observe how the workload is distributed across your key space and respective tables and indexes.

**[Introspection tables](/spanner/docs/introspection)** , primarily its system of `  Spanner_SYS  ` tables, provide a wealth of information about the database's internal state and performance. These tables expose detailed statistics on query execution, transaction behavior, lock contention, and schema details. During a POC load test, these introspection tables provide a data-driven approach to performance diagnostics, beyond what the previously mentioned insights tools offers. For example, you can use them to troubleshoot the root cause of otherwise hard to detect [lock conflicts in your database](/spanner/docs/introspection/lock-statistics#apply_best_practices_to_reduce_lock_contention) and derive actionable insights for optimization.

## Optimization

Load testing is a critical step in identifying performance issues and potential bottlenecks in your Spanner implementation. The insights gained from these tests should guide optimization efforts across schema design, transaction behavior, and query performance to ensure Spanner meets the demands of your workload.

### Optimize schema design

While an initial schema design is informed by best practices for scalability and performance, executing workloads under real-world conditions often reveals areas requiring refinement. Load tests provide valuable insights into how the schema performs under specific conditions, highlighting issues like hotspotting, uneven data distribution, or inefficiencies in query performance.

Optimization focuses on fine-tuning the following areas to align with your application's workload characteristics.

  - **Primary key adjustments:** if load tests reveal hotspots or imbalanced data distribution, review the [primary key design](/spanner/docs/schema-design#primary-key-prevent-hotspots) . Consider, for example, adding randomness in the key prefix to distribute data more evenly across nodes while preserving query efficiency.
  - **Index refinements:** load tests can reveal whether redundant indexes or over-indexing is adversely affecting write throughput. Remove unnecessary indexes or restructure existing ones for better query performance. Evaluate index selectivity and ensure they align with typical query patterns.
  - **Interleaved tables and hierarchies:** analyze whether related tables can benefit from [table interleaving](/spanner/docs/schema-and-data-model#create-interleaved-tables) to reduce query latency. Adjust interleaving decisions based on the access patterns observed during testing. Conversely, consider modeling those tables separately if the hierarchical structure causes unexpected overhead.

For information about building scalable schemas, see the [Spanner schema design best practices](/spanner/docs/schema-design) .

### Optimize transaction semantics and queries

Load tests often highlight inefficiencies in transaction and query execution, such as high contention or locking issues. Optimize transaction semantics and query structures to maximize throughput and minimize latency:

  - **Transaction modes:** use the appropriate [transaction mode](/spanner/docs/reference/rest/v1/TransactionOptions) for each workload operation. For example, use read-only transactions for queries that don't modify data, or partitioned DML for bulk updates and deletes.
  - **Batching:** where possible, use [batch write operations](/spanner/docs/batch-write) to reduce the overhead incurred by multiple round-trips.
  - **Query optimization:** refactor queries to include only necessary columns and rows, take advantage of indexes, and use query parameters in your application to reduce overhead.

For information about optimization strategies, see the [Transactions overview](/spanner/docs/transactions) and [SQL best practices](/spanner/docs/sql-best-practices) .

### Iterative load testing

Optimization is an iterative process. Conduct load tests after every significant schema or query change to validate improvements and ensure that no new bottlenecks are introduced.

Simulate realistic application scenarios with varying levels of concurrency, transaction types, and data volumes to confirm that Spanner performs as expected under peak and steady-state conditions.

### Key metrics to monitor

Track key metrics such as latency (p50, p99), throughput, and CPU utilization during optimization.

## What's next

  - Watch [How to plan and run a Spanner POC](https://www.youtube.com/watch?v=TwpzMFsdKe8) to learn the essential steps, best practices, and tooling you need to effectively evaluate Spanner's capabilities.
