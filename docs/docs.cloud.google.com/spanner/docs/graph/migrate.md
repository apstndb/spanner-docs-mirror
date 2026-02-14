**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This document describes the process of migrating your data and application to Spanner Graph. We describe the migration stages and recommended tools for each stage, depending on your source database and other factors.

Migrating your graph to Spanner Graph involves the following core stages:

1.  Gather your application requirements.
2.  Design your Spanner Graph schema.
3.  Migrate your application to Spanner Graph.
4.  Test and tune Spanner Graph.
5.  Migrate your data to Spanner Graph.
6.  Validate your data migration.
7.  Configure your cutover and failover mechanism.

To optimize your schema and application for performance, you might need to iteratively design your schema, build your application, and test and tune Spanner Graph.

## Gather your application requirements

To design a schema that meets your application needs, gather the following requirements:

  - Data modeling
  - Common query patterns
  - Latency and throughput requirements

## Design your Spanner Graph schema

To learn how to design a Spanner Graph schema, see [Spanner Graph schema overview](/spanner/docs/graph/schema-overview) for basic concepts and see [Create, update, or drop a Spanner Graph schema](/spanner/docs/graph/create-update-drop-schema) for more examples. To optimize your schema for common query patterns, see [Best practices for designing a Spanner Graph schema](/spanner/docs/graph/best-practices-designing-schema) .

## Migrate your application to Spanner Graph

First read the general Spanner guidance about [migrating your application](/spanner/docs/migration-overview#migrate_your_application) and then read the guidance in this section to learn the Spanner Graph application migration guidance.

### Connect to Spanner Graph

To learn how to connect programmatically with Spanner Graph, see [Create, update, or drop a Spanner Graph schema](/spanner/docs/graph/create-update-drop-schema) and the [Spanner Graph queries overview](/spanner/docs/graph/queries-overview) .

### Migrate queries

The Spanner Graph query interface is compatible with [ISO GQL](https://www.iso.org/standard/76120.html) , and it includes additional openCypher syntax support. For more information, see [Spanner Graph reference for openCypher users](/spanner/docs/graph/opencypher-reference) .

### Migrate mutations

To migrate your application's mutation logic, you can use Spanner table mutation mechanisms. For more information, see [Insert, update, or delete Spanner Graph data](/spanner/docs/graph/insert-update-delete-data) .

## Test and tune Spanner Graph

The Spanner guidance about how to [test and tune schema and application performance](/spanner/docs/migration-overview#test_and_tune_your_schema_and_application_performance) applies to Spanner Graph. To learn Spanner Graph performance optimization best practices, see [Best practices for designing a Spanner Graph schema](/spanner/docs/graph/best-practices-designing-schema) and [Best practices for tuning Spanner Graph queries](/spanner/docs/graph/best-practices-tuning-queries) .

## Migrate your data to Spanner Graph

To move your data from a relational database, see [Migrate your data](/spanner/docs/migration-overview#migrate-your-data) .

To move data from a graph database or a non-relational database, you can persist data from the source database into files, upload the files to Cloud Storage, and then import the files using Dataflow. Recommended file formats include AVRO and CSV. For more information, see [Recommended formats for bulk migration](/spanner/docs/migration-overview#recommended_formats_for_bulk_migration) .

### Handle Constraints

If your schema has constraints defined on input tables, make sure those constraints aren't violated during data import. Constraints include the following:

  - **Foreign Keys** : A foreign key constraint might be defined for an edge's reference to a node.
  - **Interleaving** : An edge input table might be interleaved in a node input table. This interleaving defines a parent-child relationship, with the implicit constraint that the parent must exist before the child is created.

The parent in an interleaved organization and the referenced entity in the foreign key constraint must be loaded first. This means that you must first load nodes in the graph and then load the edges. When you load edges before you load the nodes that the edges connect to, you might encounter errors during the loading process that indicate certain keys don't exist.

To achieve the correct import order, use Google-provided templates to define separate Dataflow jobs for each stage and then run the jobs in sequence. For example, you might run one Dataflow job to import nodes, and then run another Dataflow job to import edges. Alternatively, you might write a [custom Dataflow job](/dataflow/docs/guides/use-beam) that manages the import sequence.

For more information about Google-provided templates, see the following:

  - [Cloud Storage Text to Spanner template](/dataflow/docs/guides/templates/provided/cloud-storage-to-cloud-spanner) .
  - [Cloud Storage Avro to Spanner template](/dataflow/docs/guides/templates/provided/avro-to-cloud-spanner) .

If you import in the wrong order, the job might fail, or only part of your data might be migrated. If only part of your data is migrated, perform the migration again.

### Improve data loading efficiency

To improve data loading efficiency, create secondary indexes and define foreign keys after you import your data to Spanner. This approach is only possible for initial bulk loading or during migration with downtime.

## Validate your data migration

After you migrate your data, perform basic queries to verify data correctness. Run the following queries on both source and destination databases to verify that the results match:

  - Count the number of nodes and edges.
  - Count the number of nodes and edges per label.
  - Compute stats (count, sum, avg, min, max) on each node and edge property.

## Configure cutover and failover mechanism

[Configure your cutover and failover mechanisms](/spanner/docs/migration-overview#validate_your_data_migration) .

## What's next

  - [Compare Spanner Graph and openCypher](/spanner/docs/graph/opencypher-reference) .
  - [Troubleshoot Spanner Graph](/spanner/docs/graph/troubleshoot) .
