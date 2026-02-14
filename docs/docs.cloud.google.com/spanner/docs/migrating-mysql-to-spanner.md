This page offers an overview of how to migrate your Online Transactional Processing (OLTP) database from MySQL to [Spanner](/spanner) . The process to migrate to Spanner might vary depending on factors like data size, downtime requirements, application code complexity, sharding schema, custom functions, and failover and replication strategies.

The Spanner migration is broken down into the following steps:

1.  Assess your migration.
2.  Migrate your schema, and translate any SQL queries.
3.  Migrate your application to use Spanner in addition to MySQL.
4.  Load sample data and optimize your performance.
5.  Migrate your data.
6.  Validate the migration.
7.  Configure cutover and fallback mechanisms.

## Assess your migration

Assessing a migration from your source MySQL database to Spanner requires evaluating your business, technical, operational, and financial needs. For more information, see [Assess your migration](/spanner/docs/assess-migration) .

## Migrate your schema

You convert your existing schema to a Spanner [schema](/spanner/docs/schema-and-data-model) using Spanner migration tool.

For more information, see [Migrate schema from MySQL overview](/spanner/docs/migrate-mysql-schema) .

## Migrate your application to use Spanner

Spanner provides a set of [Client libraries](/spanner/docs/reference/libraries) for various languages, and the ability to read and write data using Spanner-specific API calls, as well as by using [SQL queries](/spanner/docs/query-syntax) and [Data Modification Language (DML)](/spanner/docs/dml-syntax) statements. Using API calls might be faster for some queries, such as direct row reads by key, because the SQL statement doesn't have to be translated.

Spanner provides a [JDBC driver](/spanner/docs/use-oss-jdbc) for Java applications.

As part of the migration process, features not available in Spanner as mentioned previously must be implemented in the application. For example, a trigger to verify data values and update a related table would need to be implemented in the application using a read or write transaction to read the existing row, verify the constraint, then write the updated rows to both tables.

Spanner offers [read or write and read-only transactions](/spanner/docs/transactions) which ensure external consistency of your data. Additionally, read transactions can have [Timestamp bounds](/spanner/docs/timestamp-bounds) applied, where you are reading a consistent version of the data either:

  - at an exact time in the past (up to 1 hour ago).
  - in the future (where the read will block until that time arrives).
  - with an acceptable amount of bounded staleness, which will return a consistent view up to some time in the past without needing to check that later data is available on another replica. This can give performance benefits at the expense of possibly stale data.

## Load sample data to Spanner

You can load sample data in to Spanner before performing a complete data migration to test schemas, queries, and your application.

You can use the [BigQuery reverse ETL](/bigquery/docs/export-to-spanner) workflow and the [Google Cloud CLI](/sdk) to load a small amount of data in the CSV file format in to Spanner.

For more information, see [Load sample data](/spanner/docs/load-sample-data) .

To transfer your data from MySQL to Spanner, you can also export your MySQL database to a portable file format—for example, XML—and then import that data into Spanner using Dataflow.

## Migrate data to Spanner

After optimizing your Spanner schema and loading sample data, you can move your data into an empty production-sized Spanner database.

For more information, see [Live data migration from MySQL](/spanner/docs/mysql-live-data-migration) .

## Validate your data migration

As data streams into your Spanner database, you can periodically run a comparison between your Spanner data and your MySQL data to make sure that the data is consistent. You can validate consistency by querying both data sources and comparing the results.

You can use Dataflow to perform a detailed comparison over large data sets by using [join transform](https://beam.apache.org/documentation/pipelines/design-your-pipeline/#multiple-sources) . This transform takes 2 keyed data sets, and matches the values by key. The matched values can then be compared for equality. You can regularly run this verification until the level of consistency matches your business requirements.

For more information, see [Validate your data migration](/spanner/docs/data-validation) .

## Configure cutover and fallback mechanisms

You can set up cutover and fallback for MySQL using reverse replication. Cutover and fallback means you have a contingency plan of reverting to your source MySQL database if you encounter issues with Spanner.

Reverse replication is useful when you encounter unforeseen issues with Spanner and need to fall back to the original MySQL database with minimum disruption to the service. Reverse replication enables fall back by replicating data written on Spanner back on your source MySQL database.

The reverse replication flow involves the following steps, performed by the [`  Spanner to SourceDB  ` Dataflow template](https://github.com/GoogleCloudPlatform/DataflowTemplates/tree/main/v2/spanner-to-sourcedb) :

1.  Read changes from Spanner using [Spanner change streams](/spanner/docs/change-streams) .

2.  Filter the forward-migrated changes.

3.  Transform Spanner data to be compatible with your source database schema.

4.  Verify whether the source database already contains more recent data for the specified primary key.

5.  Write the data to your source database.

## What's next

  - [Best practices in schema design](/spanner/docs/schema-design) .
  - [Optimize your Spanner schema](/spanner/docs/whitepapers/optimizing-schema-design) .
  - Learn how to use [Dataflow](/dataflow/docs/how-to) for more complex situations.
