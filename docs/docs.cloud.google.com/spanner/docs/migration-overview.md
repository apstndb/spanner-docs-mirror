This section provides an overview of how to migrate your database to Spanner. The process to migrate to Spanner might vary depending on factors like your source database, data size, downtime requirements, application code complexity, sharding schema, custom functions or transformations, and failover and replication strategies.

A typical Spanner migration involves the following stages:

1.  [Assess your migration](https://docs.cloud.google.com/spanner/docs/assess-migration) .
2.  [Migrate your schema](https://docs.cloud.google.com/spanner/docs/schema-migration) .
3.  [Modify your application code](https://docs.cloud.google.com/spanner/docs/application-migration) .
4.  [Optimize your schema and application performance](https://docs.cloud.google.com/spanner/docs/optimize-schema-performance) .
5.  [Migrate your data](https://docs.cloud.google.com/spanner/docs/data-migration) .
6.  [Validate the migration](https://docs.cloud.google.com/spanner/docs/data-validation) .
7.  [Configure cutover and fallback mechanisms](https://docs.cloud.google.com/spanner/docs/cutover-fallback-mechanisms) .

Depending on the stage of the migration you're in, you might need to consult your organization's network administrator, database administrator, or application developers to complete that migration steps outlined.

You can use the previously described high-level migration stages if you want to move a one-time dump into Spanner, or complete a large-scale production migration. Depending on your use case, what you do in each of the stages can change significantly.

## Source-specific migration guides

  - MySQL: [Migrate from MySQL to Spanner](https://docs.cloud.google.com/spanner/docs/migrating-mysql-to-spanner) .
