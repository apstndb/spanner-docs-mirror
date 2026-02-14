After optimizing your Spanner schema and migrating your application, you can move your data into an empty production-sized Spanner database, and then switch over your application to use the Spanner database.

Depending on your use case, you might be able to perform a live data migration with minimal downtime, or you might require prolonged downtime to perform the data migration.

If your application can't afford a lot of downtime, consider performing a [live data migration](#live-data-migration) . If your application can handle downtime, you can consider [migrating with downtime](#downtime-data-migration) .

In a live data migration, you need to configure the network infrastructure required for data to flow between your source database, the target Spanner database, and the tools you're using to perform the data migration. You need to decide on either private or public network connectivity depending on your organization's compliance requirements. You might need your organization's network administrator to set up the infrastructure.

## Live data migration

A live data migration consists of two components:

  - Migrating the data in a consistent snapshot of your source database.
  - Migrating the stream of changes (inserts, updates and deletes) since that snapshot, referred to as change data capture (CDC).

While live data migrations help protect your data, the process involves challenges, including the following:

  - Storing CDC data while the snapshot is migrating.
  - Writing the CDC data to Spanner while capturing the incoming CDC stream.
  - Ensuring that the migration of CDC data to Spanner is faster than the incoming CDC stream.

## Migration with downtime

If your source database can export to CSV or Avro, then you can migrate to Spanner with downtime. For more information, see [Spanner import and export overview](/spanner/docs/import-export-overview) .

Migrations with downtime can be used for test environments or applications that can handle a few hours of downtime. On a live database, a migration with downtime might result in data loss.

To perform a downtime migration, consider the following high-level approach:

1.  Stop your application and generate a dump file of the data from the source database.
2.  Upload the dump file to Cloud Storage in a MySQL, PostgreSQL, Avro, or CSV dump format.
3.  Load the dump file into Spanner using Dataflow or the Spanner migration tool.

Generating multiple small dump files makes it quicker to write to Spanner, as Spanner can read multiple dump files in parallel.

When generating a dump file from the source database, keep the following in mind to generate a consistent snapshot of data:

  - Before you perform the dump, apply a read lock on the source database to prevent the data from changing during the generation of the dump file.
  - Alternatively, generate the dump file using a read replica from the source database with replication disabled.

## Source specific guides

  - MySQL: [MySQL live data migration](/spanner/docs/mysql-live-data-migration) .
