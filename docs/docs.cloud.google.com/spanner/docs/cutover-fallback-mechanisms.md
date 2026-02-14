Migrations are often time consuming and complex. It is possible that after the data migration and cutover, you might face inconsistent performance or issues.

We recommend building in fallback mechanisms to avoid significant impact in case of an error during migration, allowing you to switch back to the source database with minimal downtime.

Reverse replication lets you fallback by replicating data written on Spanner back to your source database. If you need to fallback, reverse replication lets you point your application to the source database and continue serving requests without significant downtime.

Your reverse replication process needs to do the following:

  - Handle changes in data types or content.
  - Reverse any transformations performed during the migration.
  - Push the data to the appropriate destination, taking into account sharding schemes on the source database.

Consider the following high-level approach to building a reverse replication flow:

1.  Read the changes that occur on Spanner.
2.  (Optional) Disable the forward change data capture (CDC) migration before enabling reverse replication.
3.  Since Spanner is a distributed database, capture and temporarily order all changes before writing to your source database.
4.  Write the data to your source database.
